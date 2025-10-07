# Implementation Summary

This document describes the changes made to `auto_update_scheduled.yaml` to add person presence checking, breaking changes filtering, and mobile notifications.

## Features Added

### 1. Person Presence Check
**Location:** `when_section` input (after `pause_entities`)

- **Input Name:** `person_home_entity`
- **Type:** Entity selector (person domain)
- **Description:** Select a person entity. Updates will only proceed if this person is at home.
- **Default:** Empty (disabled)
- **Behavior:** 
  - If a person entity is selected, the automation checks if that person is home before proceeding
  - If the person is not home, updates are blocked
  - A mobile notification is sent (if enabled) to inform that updates were skipped due to person absence
  - The condition is added to the global conditions section

### 2. Breaking Changes Filter
**Location:** `what_section` input (after `update_exclusions`)

- **Input Name:** `skip_breaking_changes`
- **Type:** Boolean toggle
- **Description:** When enabled, updates containing "Breaking changes" in their release notes will be skipped
- **Default:** False (disabled)
- **Behavior:**
  - Before each update is installed, both the release summary and release notes are checked for "breaking change" keywords
  - Searches in both `release_summary` and `release_notes` attributes
  - Checks for multiple variations: "breaking change", "breaking-change", and "breaking changes" (case-insensitive)
  - If breaking changes are found and the option is enabled, the update is skipped
  - Skipped updates are logged and tracked in the `skipped_updates` variable
  - This check is applied to all update types: generic, firmware, core, OS updates, and remaining updates

### 3. Mobile Notifications
**Location:** New section `mobile_notification_section` (before `telegram_section`)

- **Input Name:** `notification_mobile_enable`
- **Type:** Boolean toggle
- **Description:** Enable mobile device notifications
- **Default:** False (disabled)

- **Input Name:** `notification_mobile_device`
- **Type:** Text input
- **Description:** The notify service name (e.g., "mobile_app_phone_name")
- **Default:** Empty string

**Notification Points:**

1. **Person Not Home Notification**
   - Sent when updates are skipped because the selected person is not home
   - Title: "[Automation Name] - Skipped"
   - Message: "Updates cannot proceed - required person is not home"

2. **Reboot Notification**
   - Sent before Home Assistant reboots
   - Title: "[Automation Name] - Rebooting"
   - Message: "Home Assistant is restarting now ([reboot type])"
   - Sent 15 seconds before the actual reboot to allow message delivery

3. **Summary Notification**
   - Sent after all updates complete
   - Title: "[Automation Name] - Complete"
   - Contains:
     - List of applied updates with count
     - List of skipped updates (breaking changes) with count
     - Reboot status (performed or required but not automatic)

## Technical Implementation Details

### Variables Added
- `input_skip_breaking_changes`: Stores the breaking changes skip setting
- `input_notification_mobile_enable`: Stores mobile notification enable status
- `input_notification_mobile_device`: Stores the notify service name
- `input_person_home_entity`: Stores the selected person entity
- `skipped_updates`: Array to track updates skipped due to breaking changes
- `applied_updates`: Array to track successfully applied updates

### Conditions Added
- Person home check: Updates only proceed if selected person is home OR no person is selected
- Mobile notifications enabled check: Notifications only sent if enabled and device is configured

### Update Flow Changes
For each update type (generic, firmware, core, OS), the following sequence is now executed:
1. Calculate current update entity
2. Check for breaking changes (new)
3. Log updating
4. Install update
5. Wait for update to complete
6. Track applied updates (new)

### Anchor Pattern
The implementation follows the existing YAML anchor pattern used in the blueprint:
- `&check_breaking_changes`: Reusable breaking changes check
- `&send_mobile_notification`: Reusable mobile notification sender (defined but direct calls used for specific messages)

## Usage Examples

### Example 1: Basic Person Presence
```yaml
person_home_entity: person.john_doe
```
Updates will only run when John Doe is home.

### Example 2: Breaking Changes Protection
```yaml
skip_breaking_changes: true
```
Any update with "breaking changes" in release notes will be skipped automatically.

### Example 3: Mobile Notifications
```yaml
notification_mobile_enable: true
notification_mobile_device: mobile_app_my_phone
```
Notifications will be sent to the mobile app about skipped updates, applied updates, and reboots.

### Example 4: All Features Combined
```yaml
person_home_entity: person.john_doe
skip_breaking_changes: true
notification_mobile_enable: true
notification_mobile_device: mobile_app_my_phone
```
Updates only when John is home, skip breaking changes, and send all notifications to mobile.

## Compatibility Notes

- All new features are optional and disabled by default
- Existing automations will continue to work without modification
- The breaking changes check uses both `release_summary` and `release_notes` attributes from update entities for comprehensive detection
- Mobile notifications use the standard Home Assistant notify service pattern
- Person state check uses the standard "home" state for person entities
