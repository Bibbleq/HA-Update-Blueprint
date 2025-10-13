# Changelog v2025.10.6

## Summary
This release fixes a critical bug where backups were not being created even when `backup_bool` was set to `true`. The issue was caused by the helper entity being set to 'on' too early in the automation flow, which prevented the backup creation condition from passing.

## 🐛 Critical Bug Fix

### Fix: Backup Not Created Due to Helper Entity Timing Issue

**Issue:** When the `update_process_started_entity` (helper entity) was configured, backups were not being created even though `backup_bool` was set to `true`. This occurred regardless of the `max_backup_age` setting.

**Root Cause:** 
The helper entity was being set to 'on' too early in the automation sequence (line 1017-1026), before the backup creation check. The backup creation logic then checked if the helper entity was in state 'off' (line 1114-1115) to determine if a backup should be created. Since the helper had just been set to 'on', this condition failed, preventing the backup from being created.

**Problematic Flow:**
1. Line 1024: Helper entity set to 'on' (when schedule is 'on' and helper is configured)
2. Line 1108-1116: Backup creation condition checks if helper is 'off' OR not configured
3. Since helper was just set to 'on', condition fails
4. Backup is NOT created ❌
5. Updates proceed without backup ❌

**Fix:**
Removed the early helper entity setting (lines 1017-1026). The helper entity should only be set to 'on' AFTER the backup is created, which is already correctly handled inside the backup creation block (line 1121).

**Correct Flow After Fix:**
1. Backup creation condition checks if helper is 'off' OR not configured ✅
2. If condition passes: Create backup
3. AFTER backup starts: Set helper to 'on' (line 1121) ✅
4. Updates proceed with backup created ✅

**Impact:** 
- ✅ Backups will now be created correctly when `backup_bool` is true
- ✅ Helper entity timing is now correct - set to 'on' only after backup creation begins
- ✅ No configuration changes required
- ✅ Works correctly with or without helper entity configured

**Related Issues:**
- This bug was introduced when the helper entity logic was added/modified
- The helper entity is used to track whether backups have been created for resume-after-restart scenarios
- The early setting was likely intended to mark the update process as started, but it interfered with the backup creation logic

## Testing Recommendations

### Test Scenario 1: With Helper Entity Configured

**Setup:**
- Configure `update_process_started_entity` with an `input_boolean` helper
- Ensure helper entity is in 'off' state
- Set `backup_bool: true`
- Set `max_backup_age: 0` (disabled)
- Have updates available

**Expected Results:**
- ✅ Backup should be created
- ✅ Helper entity should be set to 'on' AFTER backup creation begins
- ✅ Updates should proceed normally
- ✅ Helper entity should be set to 'off' after updates complete

### Test Scenario 2: Without Helper Entity Configured

**Setup:**
- Don't configure `update_process_started_entity` (leave empty)
- Set `backup_bool: true`
- Have updates available

**Expected Results:**
- ✅ Backup should be created
- ✅ Updates should proceed normally
- ✅ No errors related to helper entity

### Test Scenario 3: Resume After Restart (Helper Entity Already 'on')

**Setup:**
- Configure `update_process_started_entity`
- Manually set helper entity to 'on' state (simulating a resume scenario)
- Set `backup_bool: true`
- Have updates available

**Expected Results:**
- ✅ Backup should be SKIPPED (already created in previous run)
- ✅ Updates should proceed normally
- ✅ This demonstrates the helper entity is working correctly for resume scenarios

### Test Scenario 4: With max_backup_age Enabled

**Setup:**
- Configure `update_process_started_entity`
- Ensure helper entity is in 'off' state
- Set `backup_bool: true`
- Set `max_backup_age` to a non-zero value (e.g., 1 day)
- Ensure a recent backup exists
- Have updates available

**Expected Results:**
- ✅ Backup age check should pass (recent backup exists)
- ✅ Backup should be created
- ✅ Updates should proceed normally

## Migration Guide

No migration required! This is a pure bug fix that corrects the automation behavior to work as originally intended.

If you previously worked around this issue by:
- Disabling the helper entity
- Manually triggering backups
- Or any other workaround

You can now remove those workarounds and use the automation as designed.

## Compatibility

- ✅ Fully backward compatible
- ✅ No configuration changes needed
- ✅ Existing automations will now work correctly
- ✅ No changes to input definitions or external APIs

## Files Modified

- `auto_update_scheduled.yaml` - Removed lines 1017-1026 (early helper entity setting)
- `changelogs/CHANGELOG_v2025.10.6.md` - This changelog (new)

## Technical Details

### Line Changes

**Removed (lines 1017-1026):**
```yaml
- alias: Set ON helper flag
  if:
    - '{{ input_update_process_started_entity | string | length > 0 }}'
    - condition: state
      entity_id: !input schedule_entity
      state: 'on'
  then:
    - action: input_boolean.turn_on
      target:
        entity_id: !input update_process_started_entity
```

**Kept (inside backup creation block, line ~1121):**
```yaml
- alias: "Set helper entity to ON (if configured)"
  if:
    - '{{ input_update_process_started_entity | string | length > 0 }}'
  then:
    - action: input_boolean.turn_on
      target:
        entity_id: "{{ input_update_process_started_entity }}"
      continue_on_error: true
```

### Why This Fix Works

The helper entity serves two purposes:
1. **Track backup creation**: Prevent duplicate backups in the same update window
2. **Enable resume after restart**: Allow automation to skip backup if resuming after a system restart

The timing of when the helper is set to 'on' is critical:
- ✅ **Correct**: Set to 'on' AFTER backup creation begins (ensures backup is created first)
- ❌ **Incorrect**: Set to 'on' BEFORE backup check (prevents backup from being created)

The fix removes the early setting and relies on the existing correct setting inside the backup creation block.

## Credit

Thanks to the user who reported this issue, providing clear details about their configuration settings that helped identify the root cause.
