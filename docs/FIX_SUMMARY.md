# Fix Summary: Backup Not Created (v2025.10.6)

## Problem Statement

Users reported that backups were not being created even when `backup_bool` was set to `true`, causing updates to be installed without backup protection.

## Root Cause

The helper entity (`update_process_started_entity`) was being set to 'on' too early in the automation flow, which interfered with the backup creation logic.

## Visual Explanation

### ❌ Before Fix (INCORRECT)

```
┌─────────────────────────────────────────────────┐
│ Automation Starts                               │
│ Variables calculated:                           │
│   is_resume_after_restart = false               │
│   (helper is 'off' at start)                    │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Report list of updates                          │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ ❌ Set helper entity to 'on'                    │ ← BUG: Too early!
│    (lines 1017-1026 - REMOVED)                  │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Pre-update actions                              │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Backup section:                                 │
│   Check conditions:                             │
│   - backup_bool = true ✅                       │
│   - helper not configured OR helper is 'off'    │
│     → Helper IS configured AND is 'on' ❌       │
│   CONDITION FAILS!                              │
│   → Backup NOT created ❌                       │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Updates proceed WITHOUT backup ❌               │
└─────────────────────────────────────────────────┘
```

### ✅ After Fix (CORRECT)

```
┌─────────────────────────────────────────────────┐
│ Automation Starts                               │
│ Variables calculated:                           │
│   is_resume_after_restart = false               │
│   (helper is 'off' at start)                    │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Report list of updates                          │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ ✅ Helper entity remains 'off'                  │ ← FIX: Don't set early
│    (lines 1017-1026 removed)                    │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Pre-update actions                              │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Backup section:                                 │
│   Check conditions:                             │
│   - backup_bool = true ✅                       │
│   - helper not configured OR helper is 'off'    │
│     → Helper IS configured AND is 'off' ✅      │
│   CONDITION PASSES!                             │
│   → Create backup ✅                            │
│   → Set helper to 'on' AFTER backup starts ✅   │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ Updates proceed WITH backup ✅                  │
└─────────────────────────────────────────────────┘
```

## Code Changes

### Removed (Lines 1017-1026)

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

**Why this was wrong:** This set the helper to 'on' before the backup condition check, causing the condition to always fail when a helper entity was configured.

### Kept (Inside Backup Block, Lines ~1117-1124)

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

**Why this is correct:** This sets the helper to 'on' AFTER the backup condition passes and INSIDE the backup creation block, ensuring the helper is only set when a backup is actually being created.

## Helper Entity Purpose

The helper entity serves two important purposes:

1. **Track backup creation**: Prevent duplicate backups within the same update window
2. **Enable resume after restart**: Allow automation to resume after Home Assistant restarts without creating another backup

The timing of when the helper is set to 'on' is critical:
- ✅ **Correct**: Set to 'on' AFTER backup creation begins
- ❌ **Incorrect**: Set to 'on' BEFORE backup condition check

## Impact

### Who Was Affected
Users who:
- Had `backup_bool: true` configured
- Had a helper entity configured
- Expected backups before updates

### Who Was NOT Affected
Users who:
- Did not configure a helper entity
- Had `backup_bool: false`
- Were using What-If mode (which skips actual backup creation)

## Testing

To verify the fix works:

1. **Setup**: Configure automation with:
   - `backup_bool: true`
   - `update_process_started_entity: input_boolean.update_in_progress`
   - Ensure helper is in 'off' state
   - Have updates available

2. **Run**: Enable the schedule and let automation run

3. **Verify**:
   - ✅ Backup should be created (check logs for "Backing up Home Assistant")
   - ✅ Helper should be set to 'on' AFTER backup creation begins
   - ✅ Updates should proceed normally
   - ✅ Helper should be set to 'off' when automation completes

See [TESTING.md](TESTING.md) for comprehensive test scenarios.

## Related Files

- **auto_update_scheduled.yaml**: Main blueprint file (fix applied)
- **CHANGELOG_v2025.10.6.md**: Detailed changelog with technical explanation
- **TESTING.md**: Comprehensive testing guide
- **README.md**: Updated with fix information

## Version History

- **v2025.10.3**: Fixed backup creation when helper entity not configured
- **v2025.10.6**: Fixed backup creation when helper entity IS configured (this fix)

## Questions?

If you have questions about this fix or experience issues, please:
1. Review the [TESTING.md](TESTING.md) guide
2. Check the [CHANGELOG_v2025.10.6.md](changelogs/CHANGELOG_v2025.10.6.md)
3. Open an issue on GitHub with:
   - Your automation configuration
   - Helper entity state before/during/after run
   - Automation trace/logs
   - Expected vs actual behavior
