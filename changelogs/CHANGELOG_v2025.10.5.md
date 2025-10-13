# Changelog v2025.10.5

## Summary
This release fixes two critical issues with What-If Mode introduced in v2025.10.4:
1. **Infinite Loop**: What-If mode caused the automation to loop indefinitely
2. **Updates Still Installing**: Updates were still being installed despite What-If mode being enabled

## 🐛 Critical Bug Fixes

### Fix #1: Infinite Loop in What-If Mode

**Issue:** When What-If mode was enabled, the automation would get stuck in an infinite loop, repeating the same updates thousands of times.

**Root Cause:** 
- In What-If mode, update entities remain in state 'on' because no actual update is performed
- In normal mode, entities change to state 'off' after successful update, preventing re-processing
- The `processed_updates` variable was only being updated in What-If mode and breaking changes cases
- In normal mode, successful updates did NOT add the entity to `processed_updates`
- This caused a mismatch where What-If mode tracked processed entities but they still appeared as 'on'

**Fix:**
- Added `processed_updates` tracking to normal mode success path (line 1306)
- Added `processed_updates` tracking to normal mode timeout path (line 1314)
- This ensures all processed entities are tracked consistently regardless of success/failure or mode

**Impact:**
- ✅ What-If mode no longer loops infinitely
- ✅ All update modes now consistently track processed entities
- ✅ No behavior change for normal mode (entities still transition to 'off' state)

### Fix #2: Updates Still Installing in What-If Mode

**Issue:** Despite What-If mode being enabled, updates were still being installed, defeating the purpose of the dry-run feature.

**Root Cause:**
- The "Update - Remaining" section (line 1513-1539) is a catch-all that installs any remaining updates
- This section did NOT check for What-If mode before calling `update.install`
- While the main update loops respected What-If mode, this catch-all section bypassed it

**Fix:**
- Wrapped the "Update - Remaining - Install" service call in What-If mode conditional (line 1515)
- Added `if: '{{ not input_whatif_mode }}'` check before the update.install service call
- This ensures the catch-all section also respects What-If mode

**Impact:**
- ✅ Updates are no longer installed in What-If mode
- ✅ What-If mode now works as documented - true dry run with no changes
- ✅ Normal mode behavior unchanged

## Files Modified

- `auto_update_scheduled.yaml` - Blueprint with bug fixes for What-If Mode
- `changelogs/CHANGELOG_v2025.10.5.md` - This changelog (new)

## Technical Details

### Modified Sections

1. **Line 1306**: Normal mode success path
   ```yaml
   # Added:
   processed_updates: '{{ processed_updates + [current_update_entity] }}'
   ```

2. **Line 1314**: Normal mode timeout path
   ```yaml
   # Added:
   processed_updates: '{{ processed_updates + [current_update_entity] }}'
   ```

3. **Line 1515**: Update - Remaining - Install
   ```yaml
   # Before:
   - alias: "Update - Remaining - Install"
     continue_on_error: true
     service: update.install
     ...
   
   # After:
   - alias: "Update - Remaining - Install"
     continue_on_error: true
     if: '{{ not input_whatif_mode }}'
     then:
       - service: update.install
         ...
   ```

## Migration Guide

No migration required! These are bug fixes that don't change the configuration or behavior for properly functioning scenarios.

### To Verify Fix Works

1. Enable What-If mode:
   ```yaml
   whatif_mode: true
   ```

2. Run the automation with updates available

3. **Expected behavior:**
   - ✅ Automation completes without looping
   - ✅ No updates are installed (entities remain in 'on' state)
   - ✅ Logs show "[WHAT-IF]" prefixed messages
   - ✅ All notifications received with "[WHAT-IF]" indicators
   - ✅ No backup created
   - ✅ No system restart performed

## Testing Recommendations

### Test Scenario: What-If Mode with Multiple Updates

**Setup:**
- `whatif_mode`: true
- `backup_bool`: true
- `skip_breaking_changes`: true
- Multiple updates available (at least 3-5 updates)
- Mobile notifications enabled

**Expected Results:**
- ✅ Automation completes in reasonable time (not infinite loop)
- ✅ No backups created
- ✅ No updates installed (all entities remain state 'on')
- ✅ Log shows all updates would be processed
- ✅ Updates with breaking changes shown as "would be skipped"
- ✅ Regular updates shown as "would be updated"
- ✅ No system restart occurs
- ✅ Notifications show "[WHAT-IF]" prefix
- ✅ Summary shows what would happen without making changes

### Test Scenario: Normal Mode Verification

**Setup:**
- `whatif_mode`: false (or omit)
- Same other configuration as above

**Expected Results:**
- ✅ Automation completes normally
- ✅ Backups created (if configured)
- ✅ Updates installed as expected
- ✅ System restart occurs if needed
- ✅ Normal notifications without "[WHAT-IF]" prefix

## Known Limitations

None. What-If Mode now works correctly with all existing features:
- Breaking changes detection
- Update exclusions
- Person presence checking
- Pause entities
- All notification methods
- All update types (generic, firmware, core, OS)

---

**Last Updated:** October 2025 (v2025.10.5)
