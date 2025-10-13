# Changelog v2025.10.4

## Summary
This release introduces the What-If Mode (dry run) feature, allowing users to safely test the automation configuration without making any actual changes to their Home Assistant system.

## ✨ New Feature: What-If Mode

### Overview
What-If Mode is a comprehensive dry-run feature that simulates the entire update process without executing any actual changes. This allows users to:
- Preview which updates would be installed
- See which updates would be skipped (e.g., due to breaking changes)
- Receive all notifications with clear "[WHAT-IF]" indicators
- Validate automation configuration safely
- Build confidence before enabling production mode

### How to Use
Add `whatif_mode: true` to your automation configuration:

```yaml
automation:
  - use_blueprint:
      path: auto_update_scheduled.yaml
      input:
        schedule_entity: schedule.updates_schedule
        whatif_mode: true  # Enable dry-run mode
        skip_breaking_changes: true
        backup_bool: true
```

### What Happens in What-If Mode

**✅ Actions Performed:**
- Logs all planned actions with "[WHAT-IF]" prefix
- Lists all available updates
- Checks for breaking changes
- Sends notifications with What-If indicators
- Generates complete summary of planned actions
- Adds updates to applied/skipped lists for reporting

**❌ Actions NOT Performed:**
- No backups created
- No updates installed
- No system restarts
- No state changes to Home Assistant
- No helper entities toggled

### Example Output

```
[WHAT-IF MODE] Home Assistant Auto-update is starting
List of updates:
  - Home Assistant Core 2024.10.2
  - ESPHome 2024.10.1
  - HACS Integration

[WHAT-IF] Would backup Home Assistant
[WHAT-IF] Backup skipped in What-If mode

[WHAT-IF] Would update `Home Assistant Core 2024.10.2`...
`Home Assistant Core 2024.10.2` would be updated successfully

Skipping `ESPHome 2024.10.1` - contains breaking changes

[WHAT-IF] Would update `HACS Integration`...
`HACS Integration` would be updated successfully

[WHAT-IF] Would restart Home Assistant (core)

[WHAT-IF] Dry run complete - no actual changes made
```

## 📝 Implementation Details

### Modified Sections
1. **Blueprint Input**
   - Added `whatif_mode` boolean input (default: `false`)
   - Clear description explaining dry-run functionality

2. **Update Installation Logic**
   - Modified `&log_updating` anchor to show "[WHAT-IF]" prefix
   - Wrapped `&update_install` to skip actual update.install service calls
   - Updated `&update_wait` to handle both What-If and normal modes
   - Changes apply to all update types: generic, firmware, core, and OS

3. **Backup Logic**
   - Wrapped backup service calls in conditional check
   - Shows "[WHAT-IF]" prefix in log messages
   - Logs when backup is skipped in What-If mode

4. **Restart Logic**
   - Prevents actual system restarts in What-If mode
   - Shows what type of restart would be performed
   - Skips mobile notifications about restart

5. **Notification Messages**
   - Starting message: "[WHAT-IF MODE]" prefix
   - Summary title: "What-If Complete" vs "Complete"
   - Summary body: "Would apply updates" vs "Applied updates"
   - Done message: Clarifies no changes were made

## 📚 Documentation Updates

### README.md
- Added What-If Mode as first safety feature in Features section
- Added `whatif_mode` parameter to configuration examples
- Created new "Example 4: What-If Mode (Testing)" usage example
- Added "Scenario 4: What-If Mode Testing" test scenario
- Updated Tips & Best Practices to recommend testing with What-If mode

### Coverage
- All update types covered: generic, firmware, core, OS
- All notification channels: mobile, Telegram, logbook
- Complete simulation of the update workflow

## 🔄 Backward Compatibility

- ✅ Default value is `false` - existing automations work unchanged
- ✅ No breaking changes to existing functionality
- ✅ Purely additive feature
- ✅ No migration required for existing users

## 🧪 Testing Recommendations

### Test Scenario: What-If Mode
**Setup:**
- `whatif_mode`: true
- `backup_bool`: true
- `skip_breaking_changes`: true
- Multiple updates available (some with breaking changes)
- Mobile notifications enabled

**Expected:**
- ✅ No backup is created (but log shows it would be)
- ✅ No updates are installed (but log shows what would be)
- ✅ Updates with breaking changes shown as "would be skipped"
- ✅ Regular updates shown as "would be updated"
- ✅ No system restart occurs (but log shows it would if needed)
- ✅ Notifications show "[WHAT-IF]" prefix
- ✅ Summary shows what would happen without making changes

## 💡 Use Cases

1. **First-Time Setup**
   - Enable What-If mode before first production run
   - Review what would happen
   - Adjust configuration as needed
   - Disable What-If mode for production

2. **Configuration Changes**
   - Made changes to exclusions or settings
   - Enable What-If mode temporarily
   - Verify new behavior
   - Disable What-If mode

3. **Monthly Review**
   - Enable What-If mode
   - Check pending updates
   - Review breaking changes
   - Decide which updates to allow

4. **Sharing Configurations**
   - Recommend users enable What-If mode first
   - Let them explore behavior safely
   - Build confidence before production use

## 📊 Benefits

### For Users
1. **Safe Testing:** Test automation without risk
2. **Preview Updates:** See exactly what's pending
3. **Breaking Changes Discovery:** Identify problematic updates before installation
4. **Configuration Validation:** Verify exclusions and settings work correctly
5. **Confidence Building:** Understand behavior before production deployment

### For Developers
1. **Minimal Changes:** Leveraged existing YAML anchors for efficiency
2. **Consistent Behavior:** All update types handled uniformly
3. **Clear Indicators:** Visual prefixes make mode obvious
4. **Backward Compatible:** Default value maintains existing behavior

## 📈 Quality Metrics

- Lines changed in blueprint: ~170
- Lines changed in README: ~60
- New input added: 1
- Update types covered: 4 (generic, firmware, core, OS)
- Notification points updated: 6+
- Test scenarios added: 1
- Usage examples added: 1

## Files Modified

- `auto_update_scheduled.yaml` - Blueprint with What-If Mode logic
- `README.md` - Comprehensive documentation
- `changelogs/CHANGELOG_v2025.10.4.md` - This changelog (new)

## Migration Guide

No migration required! This is a purely additive feature.

### To Enable What-If Mode
Simply add to your automation configuration:
```yaml
whatif_mode: true
```

### To Disable What-If Mode
Either omit the setting or explicitly set:
```yaml
whatif_mode: false
```

## Known Limitations

None. What-If Mode works with all existing features:
- Breaking changes detection
- Update exclusions
- Person presence checking
- Pause entities
- All notification methods
- All update types

---

**Last Updated:** October 2025 (v2025.10.4)
