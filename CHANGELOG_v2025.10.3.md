# Changelog v2025.10.3

## Summary
This release fixes two critical issues related to backup creation and breaking changes detection.

## 🔴 Breaking Change

### Skip Breaking Changes Now Enabled by Default

**Previous Behavior:** Updates with breaking changes were processed automatically unless you explicitly enabled the `skip_breaking_changes` option.

**New Behavior:** Updates with breaking changes are now skipped by default for safety.

**Migration Required:** If you want to continue automatically applying all updates including those with breaking changes, you must explicitly set:
```yaml
skip_breaking_changes: false
```

**Rationale:** 
- Consistency with other safety features (`backup_bool` is also `true` by default)
- Safer default for automated update systems
- Breaking changes often require manual intervention
- Aligns with user expectations

## 🐛 Bug Fixes

### Fix #1: Backup Not Created When Helper Entity Not Configured

**Issue:** When the `update_process_started_entity` (helper entity) was not configured, backups were not being created even though `backup_bool` was set to `true`.

**Root Cause:** The backup condition checked if the helper entity was in state 'off', but when the entity wasn't configured (empty), this check failed and prevented backup creation.

**Fix:**
- Modified backup condition to check if entity is configured first
- If entity NOT configured: Always create backup (when backup_bool is true)
- If entity IS configured: Only create backup if entity state is 'off'
- Helper entity operations (turn on/off) only execute if entity is actually configured

**Impact:** 
- ✅ Backups will now be created even without a helper entity configured
- ✅ Existing automations with helper entities will continue to work correctly
- ✅ No configuration changes required

### Fix #2: Breaking Changes Default Changed to Enabled

**Issue:** Updates with "⚠️ Breaking Changes" were being processed automatically because the protection feature was disabled by default.

**Fix:**
- Changed `skip_breaking_changes` default from `false` to `true`
- Updated description to clarify the new default behavior
- Added migration notice in blueprint description

**Impact:**
- ✅ Breaking changes are now skipped by default (safer behavior)
- ⚠️ **Breaking Change:** Users who want all updates applied must explicitly disable this

## Technical Details

### Changes to Entity Checking Logic

Fixed multiple instances where the blueprint incorrectly treated the `update_process_started_entity` input as a list when it's actually a single entity string:

**Before:**
```yaml
{{ input_update_process_started_entity | default([]) | count == 1 }}
{{ input_update_process_started_entity | first }}
{{ input_update_process_started_entity[0] }}
```

**After:**
```yaml
{{ input_update_process_started_entity | string | length > 0 }}
{{ input_update_process_started_entity }}
```

This pattern matches the fix for `person_home_entity` in the previous release.

### Modified Sections

1. **Line 677:** Condition check for resume after restart
2. **Line 734-735:** Variable calculation for `is_resume_after_restart`
3. **Line 983:** Helper entity configuration check
4. **Line 1085-1099:** Backup creation condition (key fix)
5. **Line 1660:** Helper entity configuration check

## Testing Recommendations

### Test Scenario 1: No Helper Entity Configured
**Setup:**
- Don't configure `update_process_started_entity`
- Set `backup_bool: true`
- Have updates available

**Expected:**
- ✅ Backup should be created
- ✅ Updates should proceed normally

### Test Scenario 2: Helper Entity Configured
**Setup:**
- Configure `update_process_started_entity`
- Set `backup_bool: true`
- Ensure helper entity is 'off'
- Have updates available

**Expected:**
- ✅ Backup should be created
- ✅ Helper entity should be set to 'on'
- ✅ Updates should proceed normally

### Test Scenario 3: Updates with Breaking Changes
**Setup:**
- Leave `skip_breaking_changes` at default (now `true`)
- Have an update available with breaking changes in release notes

**Expected:**
- ✅ Update should be skipped
- ✅ Notification should be sent (if configured)
- ✅ Update should be listed in skipped_updates

### Test Scenario 4: Opt-Out of Breaking Changes Protection
**Setup:**
- Set `skip_breaking_changes: false`
- Have an update available with breaking changes

**Expected:**
- ✅ Update should be applied normally
- ✅ No special handling for breaking changes

## Migration Guide

### If You Want All Updates Applied (Including Breaking Changes)

Add this to your automation configuration:
```yaml
skip_breaking_changes: false
```

### If You Want the New Safe Defaults

No changes needed! The new defaults are:
- ✅ Backups are created before updates (`backup_bool: true`)
- ✅ Breaking changes are skipped (`skip_breaking_changes: true`)

## Compatibility

- All changes maintain backward compatibility except for the `skip_breaking_changes` default
- Existing automations will continue to work but may start skipping updates with breaking changes
- No changes to input definitions or external APIs
- Only internal logic corrections and default value change

## Files Modified

- `auto_update_scheduled.yaml` - Main blueprint file
- `IMPLEMENTATION_SUMMARY.md` - Feature documentation
- `CHANGELOG_v2025.10.3.md` - This changelog (new)
