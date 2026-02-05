# Testing Guide for HA-Update-Blueprint

This document provides test scenarios to verify the backup functionality works correctly.

## Test Setup Requirements

- Home Assistant 2024.8.0 or newer
- Schedule entity configured
- Updates available for testing
- (Optional) Helper entity (input_boolean) for testing helper entity scenarios

## Critical Test Scenarios

### Test 1: Backup Creation with Helper Entity Configured

**Purpose**: Verify that backups are created when helper entity is configured and in 'off' state.

**Setup:**
1. Create an `input_boolean` helper entity (e.g., `input_boolean.update_in_progress`)
2. Ensure the helper is in 'off' state
3. Configure automation with:
   - `backup_bool: true`
   - `update_process_started_entity: input_boolean.update_in_progress`
   - `max_backup_age: 0` (disabled)
4. Ensure at least one update is available
5. Enable the schedule

**Expected Results:**
- ✅ Automation starts
- ✅ Backup is created
- ✅ Helper entity is set to 'on' AFTER backup creation begins
- ✅ Updates are installed
- ✅ Helper entity is set to 'off' when automation completes

**How to Verify:**
1. Check Home Assistant logs for "Backing up Home Assistant" message
2. Verify backup appears in Settings > System > Backups
3. Monitor helper entity state changes in Developer Tools > States
4. Check automation trace to see the sequence of actions

**If Test Fails:**
- If backup is not created, check that helper entity was in 'off' state before automation started
- If helper entity never changes to 'on', check helper entity is properly configured
- If backup is created but updates don't proceed, check for other conditions (schedule, pause entities, etc.)

---

### Test 2: Backup Creation without Helper Entity

**Purpose**: Verify that backups are created when no helper entity is configured.

**Setup:**
1. Configure automation with:
   - `backup_bool: true`
   - `update_process_started_entity:` (leave empty/not configured)
   - `max_backup_age: 0` (disabled)
2. Ensure at least one update is available
3. Enable the schedule

**Expected Results:**
- ✅ Automation starts
- ✅ Backup is created
- ✅ Updates are installed
- ✅ No errors related to helper entity

**How to Verify:**
1. Check Home Assistant logs for "Backing up Home Assistant" message
2. Verify backup appears in Settings > System > Backups
3. Check automation trace shows backup creation step executed

**If Test Fails:**
- If backup is not created, check automation logs for error messages
- Verify `backup_bool` is set to `true`
- Check that schedule is 'on' and other conditions are met

---

### Test 3: Resume After Restart (No Duplicate Backup)

**Purpose**: Verify that backup is NOT created when resuming after restart (helper entity already 'on').

**Setup:**
1. Configure automation with helper entity
2. Manually set helper entity to 'on' state (simulating a previous run)
3. Ensure updates are available
4. Enable the schedule

**Expected Results:**
- ✅ Automation starts
- ✅ Backup is SKIPPED (not created again)
- ✅ Updates proceed normally
- ✅ Helper entity remains 'on' during updates
- ✅ Helper entity is set to 'off' when automation completes

**How to Verify:**
1. Check Home Assistant logs - should NOT see "Backing up Home Assistant" message
2. Verify no new backup is created in Settings > System > Backups
3. Check automation trace shows backup section was skipped due to `is_resume_after_restart` condition

**If Test Fails:**
- If backup is created (should not be), verify helper entity was actually in 'on' state before automation started
- If updates don't proceed, check other conditions (schedule, pause entities, etc.)

---

### Test 4: Backup with max_backup_age Validation

**Purpose**: Verify that backup age validation works correctly.

**Setup:**
1. Configure automation with:
   - `backup_bool: true`
   - `max_backup_age: 1 day` (or other non-zero value)
2. Ensure a recent backup exists (within the age limit)
3. Ensure `sensor.backup_last_successful_automatic_backup` exists (modern HA) or backup sensors with `last_backup` attribute exist (legacy)
4. Ensure updates are available
5. Enable the schedule

**Expected Results:**
- ✅ Automation starts
- ✅ Backup age check passes (recent backup found)
- ✅ New backup is created
- ✅ Updates proceed normally

**How to Verify:**
1. Check automation logs for backup age validation messages
2. Verify new backup is created
3. Check automation trace shows backup age check passed

**If Test Fails:**
- If automation stops with "Last backup is too old", verify a recent backup exists
- If automation stops with "No backup entity found", check that `sensor.backup_last_successful_automatic_backup` exists (modern HA) or check for sensors with `last_backup` attribute (legacy)
- Check sensor states in Developer Tools > States

---

### Test 5: Backup Disabled

**Purpose**: Verify that backups are NOT created when `backup_bool` is false.

**Setup:**
1. Configure automation with:
   - `backup_bool: false`
2. Ensure updates are available
3. Enable the schedule

**Expected Results:**
- ✅ Automation starts
- ✅ Backup is NOT created
- ✅ Updates proceed without backup

**How to Verify:**
1. Check Home Assistant logs - should NOT see "Backing up Home Assistant" message
2. Verify no new backup is created
3. Updates should still be installed

---

## Debugging Failed Tests

### Common Issues

1. **Backup not created despite backup_bool: true**
   - Check helper entity state (should be 'off' for first run)
   - Verify schedule is 'on'
   - Check `is_resume_after_restart` variable value in automation trace
   - Look for error messages in Home Assistant logs

2. **Helper entity not changing state**
   - Verify helper entity exists and is type `input_boolean`
   - Check helper entity is not disabled
   - Look for errors in automation trace

3. **Backup age check failing**
   - Verify `sensor.backup_last_successful_automatic_backup` exists (modern HA) or backup sensors exist with `last_backup` attribute (legacy)
   - Check sensor values are recent enough
   - Consider setting `max_backup_age: 0` to disable this check

### How to View Automation Trace

1. Go to Settings > Automations & Scenes
2. Find your auto-update automation
3. Click on the automation
4. Click "Traces" tab
5. Select the most recent run
6. Expand each step to see variables, conditions, and actions

### How to View Home Assistant Logs

1. Go to Settings > System > Logs
2. Search for "Auto-update" or your automation name
3. Look for error messages or warnings
4. Filter by severity if needed

## Additional Testing Notes

- **What-If Mode**: Test with `whatif_mode: true` to safely simulate the automation without making changes
- **Notifications**: Enable mobile or Telegram notifications to monitor automation progress in real-time
- **Verbose Logging**: Enable `verbose_logging_bool: true` for more detailed logs

## Version History

- **v2025.10.10**: Fixed backup age check to use modern HA backup sensor (`sensor.backup_last_successful_automatic_backup`) with fallback to legacy sensors
- **v2025.10.6**: Added tests for backup creation fix (helper entity timing issue)
- **v2025.10.5**: Fixed What-If mode infinite loop
- **v2025.10.4**: Added What-If mode feature
- **v2025.10.3**: Fixed backup creation when helper entity not configured

## Contributing

If you discover new test scenarios or edge cases, please:
1. Document the scenario
2. Include expected results
3. Submit a pull request or issue on GitHub
