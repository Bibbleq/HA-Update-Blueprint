# Home Assistant Auto-Update Blueprint

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.8.0+-blue.svg)](https://www.home-assistant.io/)
[![Blueprint](https://img.shields.io/badge/Blueprint-YAML-orange.svg)](https://www.home-assistant.io/docs/automation/using_blueprints/)

A powerful and safe Home Assistant blueprint that automatically updates Home Assistant Core, OS, add-ons, and integrations on a scheduled basis with intelligent safety features.

## 🎉 Latest Update - v2025.10.8

**Enhanced Backup Detection:**
- **Event-Driven Monitoring:** Switched from polling to event-driven backup detection for faster and more reliable completion detection
- **New Backup Service:** Using `backup.create_automatic` instead of `hassio.backup_full` for improved compatibility with HA's native backup system
- **Dual Trigger System:**
  - Primary: Monitors `sensor.backup_backup_manager_state` transition (create_backup → idle)
  - Fallback: Monitors `sensor.backup_last_successful_automatic_backup` updates
- **Better Diagnostics:** Logs backup duration and detection method used
- **No More Polling Delays:** Instant detection when backup completes, no unnecessary waiting
- **Deprecated:** `backup_location` input (now uses HA's configured backup location)

**Impact:** Backups are detected as soon as they complete, reducing automation run time and eliminating false timeout warnings. See changelog for full details.

## Previous Update - v2025.10.7

**Critical Bug Fixes:**
- **What-If Mode:** Fixed issue where What-If mode was still creating actual backups and waiting for completion. What-If mode now properly skips all backup operations and immediately continues to update simulation.
- **Backup Monitoring:** Enhanced backup completion detection with fallback methods (idle state check) to reduce false timeouts.
- **Notifications:** Added `[WHAT-IF MODE]` prefix to all relevant notifications throughout the automation for better clarity.
- **Logging:** Improved backup timeout messages to indicate the backup may still be running in the background.

**Impact:** What-If mode now works as a true dry run without creating backups or making any changes. Backup monitoring is more reliable. All notifications clearly indicate What-If mode status. See [CHANGELOG_v2025.10.7.md](changelogs/CHANGELOG_v2025.10.7.md) for full details.

## ⚠️ Important Notice

- **Home Assistant may restart automatically** as part of the update process
- **Use at your own risk!** Please review Home Assistant community discussions about the risks involved in auto-updating production systems
- **Ensure you have reliable backups** before enabling automatic updates
- Auto-updates in production environments may lead to temporary system instability
- Updates may affect connected devices and integrations

## 🌟 Features

### Safety & Control Features

- **🧪 What-If Mode (Dry Run)**
  - Test the automation safely without making any changes
  - Shows what updates would be installed and what would be skipped
  - All notifications and logging work normally
  - No backups are created, no updates installed, no restarts performed
  - Perfect for testing automation configuration before using it in production

- **🛡️ Breaking Changes Protection** (Default: Enabled)
  - Automatically detects and skips updates containing breaking changes
  - Searches both release summary and release notes for breaking change indicators
  - Case-insensitive detection of multiple variations: "breaking change", "breaking-change", "breaking changes"
  - Can be disabled for users who want all updates applied

- **💾 Automatic Backups**
  - Creates backups before applying updates (enabled by default)
  - Event-driven monitoring for instant completion detection
  - Uses Home Assistant's native automatic backup service
  - Dual trigger system (state transition + fallback)
  - Logs backup duration and detection method
  - Works with or without helper entity configuration
  - Optional control via helper entity state

- **👤 Person Presence Check**
  - Only run updates when specific person is home
  - Prevents updates during absence
  - Sends notifications when updates are skipped due to absence

- **⏸️ Flexible Pause Control**
  - Pause updates using multiple methods:
    - Individual pause entities
    - Helper entity state
    - Person presence requirement

### Update Management

- **📦 Comprehensive Update Coverage**
  - Home Assistant Core updates
  - Home Assistant OS updates
  - Add-on updates (HACS and built-in)
  - Integration updates
  - Firmware updates

- **🎯 Smart Update Control**
  - Update exclusions (skip specific entities)
  - Type-based filtering (skip specific update types)
  - Schedule-based execution
  - Priority ordering (generic → firmware → core → OS)

- **🔄 Resume After Restart**
  - Automatically resumes update process after Home Assistant restarts
  - Tracks update progress using optional helper entity
  - Handles interruptions gracefully

### Notification & Logging

- **📱 Mobile App Notifications**
  - Person not home notifications
  - Pre-reboot warnings (15 seconds before reboot)
  - Update completion summary with:
    - List of applied updates
    - List of skipped updates (breaking changes)
    - Reboot status

- **📨 Telegram Integration**
  - Full Telegram notification support
  - Customizable messages
  - Same notification points as mobile app

- **📝 Logbook Integration**
  - Detailed logging of all update activities
  - Track applied and skipped updates
  - Visibility in Home Assistant's logbook

## 📥 Installation

### Prerequisites

- Home Assistant 2024.8.0 or newer
- (Optional) Helper entity for update process tracking
- (Optional) Mobile app or Telegram for notifications

### Import Blueprint

1. **Via Home Assistant UI:**
   - Navigate to Settings → Automations & Scenes → Blueprints
   - Click the "Import Blueprint" button
   - Enter the blueprint URL: `https://github.com/Bibbleq/HA-Update-Blueprint/blob/main/auto_update_scheduled.yaml`

2. **Via URL:**
   ```
   https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Bibbleq/HA-Update-Blueprint/blob/main/auto_update_scheduled.yaml
   ```

## ⚙️ Configuration

### Basic Configuration

Minimal setup for safe automatic updates:

```yaml
# The blueprint will use these defaults:
# - skip_breaking_changes: true (SAFE)
# - backup_bool: true (SAFE)
# - No person presence check
# - No update exclusions
```

### Advanced Configuration Example

```yaml
# Schedule
schedule_entity: schedule.updates_schedule

# Safety Features
whatif_mode: false                    # Dry-run mode for testing (default: false)
skip_breaking_changes: true           # Skip updates with breaking changes (default)
backup_bool: true                     # Create backup before updates (default)
person_home_entity: person.john_doe   # Only update when John is home

# Update Control
update_exclusions:
  - update.hacs_excluded_integration
  - update.sensitive_addon
  
update_types_exclusions:
  - device_update                     # Skip firmware updates

# Helper Entity (optional)
update_process_started_entity: input_boolean.update_in_progress

# Pause Control (optional)
pause_entities:
  - input_boolean.pause_updates
  - binary_sensor.maintenance_mode

# Mobile Notifications
notification_mobile_enable: true
notification_mobile_device: mobile_app_my_phone

# Telegram Notifications (optional)
notification_telegram: true
notification_telegram_bot_token: !secret telegram_bot_token
notification_telegram_chat_id: !secret telegram_chat_id
```

## 🚀 Usage Examples

### Example 1: Maximum Safety (Recommended for Production)

```yaml
skip_breaking_changes: true
backup_bool: true
person_home_entity: person.admin
notification_mobile_enable: true
notification_mobile_device: mobile_app_admin_phone
```

**Benefits:**
- ✅ Updates only when admin is home
- ✅ Breaking changes automatically skipped
- ✅ Automatic backups before updates
- ✅ Mobile notifications for awareness

### Example 2: Aggressive Updates (Development/Testing)

```yaml
skip_breaking_changes: false
backup_bool: true
auto_reboot: true
```

**Use Case:**
- Development/testing environments
- Want all updates immediately
- Still maintains backup safety net

### Example 3: Selective Updates with Exclusions

```yaml
skip_breaking_changes: true
backup_bool: true
update_exclusions:
  - update.custom_integration_a
  - update.experimental_addon
update_types_exclusions:
  - device_update
```

**Use Case:**
- Skip specific problematic integrations
- Exclude firmware updates (manual control preferred)
- Still get core and addon updates automatically

### Example 4: What-If Mode (Testing)

```yaml
whatif_mode: true
skip_breaking_changes: true
backup_bool: true
notification_mobile_enable: true
notification_mobile_device: mobile_app_my_phone
```

**Benefits:**
- ✅ Test automation without making any changes
- ✅ See what updates would be applied
- ✅ See what would be skipped due to breaking changes
- ✅ Receive notifications showing planned actions
- ✅ No backups created, no updates installed, no restarts
- ✅ Perfect for validating configuration before production use

**Use Case:**
- Testing new automation configuration
- Validating exclusion rules
- Checking which updates are pending
- Safe exploration of update behavior

## 📋 Version History

### v2025.10.8 (Current)

**✨ Enhanced Backup Detection:**
- Switched from `hassio.backup_full` to `backup.create_automatic` for improved reliability
- Replaced polling-based monitoring with event-driven triggers
- Primary trigger: `sensor.backup_backup_manager_state` (create_backup → idle)
- Fallback trigger: `sensor.backup_last_successful_automatic_backup` updates
- Added backup duration tracking and detection method logging
- Removed polling delays for instant completion detection
- Deprecated `backup_location` input (uses HA's configured backup location)

**Impact:**
- ✅ Faster backup completion detection (no polling delays)
- ✅ More reliable monitoring with dual trigger system
- ✅ Better diagnostics with duration and method logging
- ✅ Reduced false timeout warnings
- ✅ Improved compatibility with HA's native backup system

### v2025.10.7

**🐛 Critical Bug Fixes:**
- Fixed What-If mode still creating actual backups (now properly skips backup creation)
- Enhanced backup completion monitoring with fallback detection methods (idle state check)
- Added `[WHAT-IF MODE]` prefix to all notifications for better clarity
- Improved backup timeout messages with more detail

**Impact:**
- ✅ What-If mode now works as a true dry run without creating backups
- ✅ Backup monitoring more reliable with reduced false timeouts
- ✅ All notifications clearly indicate What-If mode status
- ✅ Better logging when backup times out

See [CHANGELOG_v2025.10.7.md](changelogs/CHANGELOG_v2025.10.7.md) for detailed technical information.

### v2025.10.5

**🐛 Critical Bug Fixes:**
- Fixed infinite loop in What-If mode that caused automation to repeat indefinitely
- Fixed updates still being installed despite What-If mode being enabled
- Added `processed_updates` tracking to normal mode success and timeout paths
- Wrapped "Update - Remaining" section in What-If mode conditional

**Impact:**
- ✅ What-If mode now works correctly - no infinite loops
- ✅ No updates are installed in What-If mode (true dry run)
- ✅ All update modes now consistently track processed entities

See [CHANGELOG_v2025.10.5.md](changelogs/CHANGELOG_v2025.10.5.md) for detailed technical information.

### v2025.10.4

**✨ New Feature:**
- Added What-If Mode (dry run) for safe automation testing
- Preview updates without making any changes
- See what would be installed and what would be skipped
- All notifications work normally with "[WHAT-IF]" prefix
- No backups, updates, or restarts performed in What-If mode

**📚 Documentation:**
- Added comprehensive What-If Mode documentation
- New usage example for What-If Mode testing
- Added test scenario for What-If Mode
- Updated tips and best practices

### v2025.10.3

**🔴 Critical Fixes:**
- Fixed breaking changes detection not working correctly
- Fixed backup creation when helper entity not configured

**🔴 Breaking Changes:**
- `skip_breaking_changes` now defaults to `true` (enabled) for safety
- **Migration Required:** Set `skip_breaking_changes: false` if you want all updates

**🐛 Bug Fixes:**
- Corrected entity checking logic (treated as string instead of list)
- Fixed backup condition to work without helper entity
- Improved OR logic for backup creation

**📝 Technical Details:**
- Restructured breaking changes check to avoid nested sequences
- Removed problematic `stop` command
- Added conditional wrappers around update steps
- Fixed 5 locations where entity checking was incorrect

### v2025.10.2

**Features:**
- Added person presence checking
- Added breaking changes filter (initial implementation)
- Added mobile app notifications
- Improved backup process compatibility

## 🔄 Migration Guides

### Upgrading to v2025.10.3

#### If You Want the New Safe Defaults ✅

**No action required!** The new defaults are:
- ✅ Backups are created before updates
- ✅ Breaking changes are skipped automatically

#### If You Want All Updates Applied (Including Breaking Changes)

Add this to your automation configuration:
```yaml
skip_breaking_changes: false
```

### From Earlier Versions

All changes maintain backward compatibility except for the `skip_breaking_changes` default value change. Existing automations will continue to work but may start skipping updates with breaking changes.

## 🧪 Testing Scenarios

### Scenario 1: No Helper Entity + Breaking Changes (Default)
**Setup:**
- `update_process_started_entity`: Not configured
- `backup_bool`: true
- `skip_breaking_changes`: true

**Expected:**
- ✅ Backup is created
- ✅ Update with breaking changes is skipped
- ✅ Notification sent about skipped update

### Scenario 2: With Helper Entity + Normal Updates
**Setup:**
- `update_process_started_entity`: Configured, state is 'off'
- `backup_bool`: true
- Update available without breaking changes

**Expected:**
- ✅ Backup is created
- ✅ Helper entity set to 'on'
- ✅ Update is applied normally

### Scenario 3: Person Not Home
**Setup:**
- `person_home_entity`: Configured
- Person is away

**Expected:**
- ✅ Updates are skipped
- ✅ Mobile notification sent (if enabled)
- ✅ Automation exits gracefully

### Scenario 4: What-If Mode Testing
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

For more detailed test scenarios and debugging guides, see [TESTING.md](TESTING.md).

## 🤝 Contributing

This is a fork/modification of the original blueprint by [edwardtfn](https://github.com/edwardtfn). 

### Original Repository
- Original Author: Edward Firmo
- Original Repository: https://github.com/edwardtfn/ha_auto_update_scheduled
- Community Discussion: https://community.home-assistant.io/t/459281

### This Fork
- Repository: https://github.com/Bibbleq/HA-Update-Blueprint
- Issues & Feature Requests: https://github.com/Bibbleq/HA-Update-Blueprint/issues

## 📚 Additional Documentation

- [IMPLEMENTATION_SUMMARY.md](IMPLEMENTATION_SUMMARY.md) - Detailed technical implementation
- [changelogs/](changelogs/) - Version changelogs and release notes

## 💡 Tips & Best Practices

1. **Start Conservative:** Use the default safe settings initially
2. **Test First:** Use What-If mode (`whatif_mode: true`) to safely test your configuration before production
3. **Validate Configuration:** Run the automation in What-If mode to see what would be updated and skipped
4. **Enable Notifications:** Stay informed about update activities
5. **Review Logs:** Check Home Assistant logbook after automation runs
6. **Have Backups:** Always maintain external backups beyond the automation's built-in backup
7. **Monitor Breaking Changes:** Review what updates were skipped and apply manually when ready
8. **Use Helper Entity:** For better control and resume functionality
9. **Schedule Wisely:** Choose low-usage time windows for updates

## ⚠️ Known Limitations

- Breaking changes detection relies on keywords in release notes
- Some updates may not include proper release notes
- Network interruptions during updates can cause issues
- Restart timing may vary depending on system performance
- Mobile notifications sent 15 seconds before reboot may not always deliver in time on slower networks

## 📞 Support & Discussion

- **Issues:** [GitHub Issues](https://github.com/Bibbleq/HA-Update-Blueprint/issues)
- **Original Discussion:** [Home Assistant Community Forum](https://community.home-assistant.io/t/459281)
- **Original Repository Issues:** [edwardtfn/ha_auto_update_scheduled](https://github.com/edwardtfn/ha_auto_update_scheduled/issues)

## 📄 License

This blueprint maintains compatibility with the original project's licensing terms. Please refer to the original repository for specific license information.

## ☕ Support the Original Author

If you find this blueprint useful, consider supporting the original author:
- [Buy Edward a Coffee](https://buymeacoffee.com/edwardfirmo)

---

**Last Updated:** October 2025 (v2025.10.8)
