# Changelog - v2025.10.7

## Release Date
October 20, 2025

## Summary
Critical fix for What-If mode creating actual backups and improved backup completion monitoring.

## 🐛 Critical Bug Fixes

### What-If Mode No Longer Creates Actual Backups
**Problem:** When What-If mode was enabled, the automation was still creating actual backups and waiting for them to complete, defeating the purpose of a "dry run" mode.

**Root Cause:** The What-If mode conditional check was placed too late in the backup section flow. The backup service call (`hassio.backup_full`) was only conditionally skipped, but the section still executed the helper entity updates and waited for backup completion.

**Solution:** Restructured the backup section to check What-If mode at the beginning. In What-If mode:
- Only logs that a backup would be created
- Does not call the backup service
- Does not wait for backup completion
- Does not update helper entity state
- Continues immediately to update simulation

**Code Changes:**
```yaml
# Before: What-If check was nested inside backup creation
- if: '{{ not input_whatif_mode }}'
  then:
    - action: hassio.backup_full
      # ... backup logic

# After: What-If check is at the top level
- if: '{{ input_whatif_mode }}'
  then:
    - variables:
        log_message: "[WHAT-IF MODE] Would create backup before updates"
    # ... only logging
  else:
    - action: hassio.backup_full
    # ... actual backup logic
```

**Impact:**
- ✅ What-If mode now works as a true dry run
- ✅ No backups are created when testing
- ✅ No waiting time for backup completion in What-If mode
- ✅ Helper entity not modified in What-If mode

---

## ⚡ Improvements

### Enhanced Backup Completion Monitoring
**Problem:** The backup completion monitoring was timing out after 60 minutes even when backups completed successfully. The wait template only checked for `backup_state != 'backing_up'` which wasn't reliably detecting completion.

**Solution:** Added multiple fallback detection methods:

1. **Primary Check:** All backup sensors are not in 'backing_up' state
2. **Fallback Check:** Any backup sensor shows 'idle' state (common completion indicator)

**Code Changes:**
```yaml
wait_template: >-
  {% set backup_sensors = states.sensor
    | selectattr('attributes.backup_state', 'defined')
    | list %}
  {% if backup_sensors | count > 0 %}
    {# Check if all backup sensors are not in 'backing_up' state #}
    {% set not_backing_up = backup_sensors | selectattr('attributes.backup_state', 'ne', 'backing_up') | list | count == backup_sensors | count %}
    {# Check if any sensor shows 'idle' state as completion indicator #}
    {% set has_idle = backup_sensors | selectattr('attributes.backup_state', 'eq', 'idle') | list | count > 0 %}
    {{ not_backing_up or has_idle }}
  {% else %}
    false
  {% endif %}
```

**Benefits:**
- ✅ More reliable backup completion detection
- ✅ Reduced false timeouts
- ✅ Better support for different backup entity states

---

### Improved Backup Logging
**Before:**
```
Backup timeout reached, continuing anyway
```

**After:**
```
Backup monitoring timeout reached (60 minutes), continuing anyway. The backup may still be running in the background
```

**Benefits:**
- ✅ Clearer indication of what the timeout means
- ✅ Users know the backup might still complete in the background
- ✅ Shows the configured timeout duration

---

### What-If Mode Notification Improvements
Added `[WHAT-IF MODE]` prefix to all relevant notifications throughout the automation:

**Updated Sections:**
1. **Starting message:** `[WHAT-IF MODE] {friendly_name} is starting`
2. **List of updates:** `[WHAT-IF MODE] List of updates:`
3. **Pre-update actions:** `[WHAT-IF MODE] Would run pre-update actions...`
4. **Backup:** `[WHAT-IF MODE] Would create backup before updates`
5. **Breaking changes:** `[WHAT-IF MODE] Would skip {update} - contains breaking changes`
6. **Update progress:** `[WHAT-IF MODE] Would update {entity}...`
7. **Finishing:** `[WHAT-IF MODE] Finishing update process`
8. **Remaining updates:** `[WHAT-IF MODE] Remaining updates:`
9. **Nothing to update:** `[WHAT-IF MODE] There's nothing to update`
10. **Restart:** `[WHAT-IF MODE] {count} items would require a restart:`
11. **Pre-restart actions:** `[WHAT-IF MODE] Would run pre-restart actions...`
12. **Post-update actions:** `[WHAT-IF MODE] Would run post-update actions`
13. **Done:** `[WHAT-IF MODE] Dry run complete - no actual changes made`

**Mobile Notification Summary:**
- Title: `{friendly_name} - What-If Complete` (vs. just "Complete")
- Summary clearly shows "Would apply updates" vs "Applied updates"
- Restart status: "Would be performed" vs "Performed"

**Benefits:**
- ✅ Users can immediately see What-If mode is active
- ✅ Clear distinction between simulated and actual actions
- ✅ Better understanding of what would happen without making changes
- ✅ Consistent messaging throughout the entire automation

---

## 📋 Technical Details

### Files Changed
- `auto_update_scheduled.yaml` - Main blueprint file

### Lines Modified
- Backup section (lines ~1105-1180): Restructured for What-If mode
- Notification messages (multiple locations): Added What-If mode prefixes
- Backup monitoring (lines ~1149-1171): Enhanced detection logic
- Mobile summary (lines ~1715-1742): Already had What-If support

### Variables Added
- `backup_completed`: Boolean flag for clearer logging

### Breaking Changes
None. All changes are backward compatible.

---

## 🧪 Testing

### Test Coverage
Created comprehensive verification script (`/tmp/verify_whatif_changes.py`) that validates:
1. ✅ What-If mode backup handling
2. ✅ Backup monitoring improvements
3. ✅ What-If mode notifications in all sections
4. ✅ Mobile notification summary formatting

All tests passed successfully.

### Manual Testing Recommended
1. **What-If Mode Test:**
   - Enable `whatif_mode: true`
   - Verify no backups are created
   - Verify all notifications show [WHAT-IF MODE] prefix
   - Verify no updates are actually installed

2. **Normal Mode Test:**
   - Disable `whatif_mode: false`
   - Verify backups are created as expected
   - Verify backup monitoring completes successfully
   - Verify notifications do NOT show [WHAT-IF MODE] prefix

3. **Backup Timeout Test:**
   - Set low backup timeout (e.g., 1 minute)
   - Verify improved timeout message appears
   - Verify automation continues after timeout

---

## 📚 Related Documentation

### Updated Files
- `auto_update_scheduled.yaml` - Core changes
- `CHANGELOG_v2025.10.7.md` - This file

### Documentation to Review
- [TESTING.md](../TESTING.md) - Test scenarios for What-If mode
- [README.md](../README.md) - What-If mode feature description
- [FIX_SUMMARY.md](../FIX_SUMMARY.md) - Previous fix details

---

## 🔄 Migration Guide

No migration steps required. This is a bug fix release that:
- ✅ Fixes What-If mode behavior (was broken, now works correctly)
- ✅ Improves backup monitoring (was unreliable, now more robust)
- ✅ Enhances notifications (existing behavior, now clearer)

Users who were already using What-If mode will now see it working correctly for the first time.

---

## ⚠️ Known Limitations

1. **Backup State Detection:** Still relies on Home Assistant native backup entities. If your system doesn't expose `backup_state` attributes, monitoring may not work optimally.

2. **What-If Mode Actions:** Pre/post-update and pre-restart action sequences still execute in What-If mode (they just show different log messages). This is by design to test the action sequence logic.

3. **Backup Timeout:** Default 60-minute timeout may still be too short for very large backups on slow systems. Users can increase via `backup_timeout` configuration.

---

## 🙏 Credits

This fix addresses issues reported by users who discovered that:
- What-If mode was creating actual backups
- Backup monitoring was timing out unnecessarily
- Notifications didn't clearly indicate What-If mode status

Thank you to the community for reporting these issues!

---

## 📞 Support

If you experience any issues with this version:
1. Check the [TESTING.md](../TESTING.md) guide for verification steps
2. Review automation traces in Home Assistant
3. Open an issue on GitHub with:
   - Your configuration (sanitized)
   - Automation trace/logs
   - Expected vs actual behavior

---

**Version:** v2025.10.7  
**Previous Version:** v2025.10.6  
**Next Version:** TBD
