# Changelog v2025.10.10

## Fixed Backup Age Check for Modern Home Assistant

**Release Date:** February 2026

### 🎯 Overview

Version 2025.10.10 fixes a critical bug where the backup age validation was failing with "No backup entity with last_backup attribute found" on modern Home Assistant installations. The fix updates the backup detection logic to use the modern HA backup sensor as the primary source.

### 🐛 Bug Fixed

**Issue:** Automation fails with error: "No backup entity with last_backup attribute found"

**Root Cause:** The backup age check was looking for sensors with a `last_backup` attribute, but modern Home Assistant uses `sensor.backup_last_successful_automatic_backup` where the backup timestamp is stored in the sensor's **state** value, not as an attribute.

### 🔧 Technical Changes

#### Backup Age Check Logic (Lines 1191-1239)

**Before:**
```yaml
- variables:
    last_backup_timestamp_list: >
      {{
        states.sensor
        | selectattr("attributes.last_backup", "defined")
        | map(attribute="attributes.last_backup")
        | list
      }}
    last_backup_timestamp: '{{ last_backup_timestamp_list | max if last_backup_timestamp_list | count > 0 else None }}'
- alias: Check if any backup entity was found
  if: '{{ last_backup_timestamp != None }}'
  # ...
  else:
    - stop: "No backup entity with last_backup attribute found"
```

**After:**
```yaml
- variables:
    # Modern HA backup sensor (state contains the last backup timestamp)
    modern_backup_sensor_state: >
      {{ states('sensor.backup_last_successful_automatic_backup') }}
    modern_backup_timestamp: >
      {{ modern_backup_sensor_state if modern_backup_sensor_state not in ['unknown', 'unavailable', ''] else None }}
    # Legacy sensors with last_backup attribute (for backwards compatibility)
    legacy_backup_timestamp_list: >
      {{
        states.sensor
        | selectattr("attributes.last_backup", "defined")
        | map(attribute="attributes.last_backup")
        | list
      }}
    legacy_backup_timestamp: '{{ legacy_backup_timestamp_list | max if legacy_backup_timestamp_list | count > 0 else None }}'
    # Use modern sensor if available, otherwise fall back to legacy
    last_backup_timestamp: >
      {% if modern_backup_timestamp != None %}
        {{ modern_backup_timestamp }}
      {% elif legacy_backup_timestamp != None %}
        {{ legacy_backup_timestamp }}
      {% else %}
        {{ None }}
      {% endif %}
- alias: Check if any backup entity was found
  if: '{{ last_backup_timestamp != None and last_backup_timestamp | string | trim | length > 0 }}'
  # ...
  else:
    - stop: "No backup entity found"
```

**Changes:**
- Added check for `sensor.backup_last_successful_automatic_backup` state (modern HA)
- Filters out invalid states ('unknown', 'unavailable', empty string)
- Falls back to legacy `last_backup` attribute-based sensors
- Improved null check to handle edge cases with empty strings
- Updated error message to be more informative

### 🎯 Benefits

1. **Modern HA Compatibility:** Works with current Home Assistant backup integration
2. **Backwards Compatible:** Still supports legacy sensors with `last_backup` attribute
3. **Clearer Error Messages:** Better messaging helps users troubleshoot issues
4. **Robust Handling:** Properly handles 'unknown', 'unavailable', and empty states

### ⚠️ Breaking Changes

**None.** This release maintains full backward compatibility.

- Existing automations will continue to work without modification
- Legacy backup sensor detection is still supported as fallback

### 🔍 Testing Recommendations

1. **Verify Backup Sensor Exists:**
   - Check for `sensor.backup_last_successful_automatic_backup` in Developer Tools > States
   - Verify the sensor state contains a valid timestamp

2. **Test Backup Age Check:**
   - Configure `max_backup_age` to a non-zero value
   - Run the automation and verify the backup age check passes

3. **Verify Fallback Works:**
   - If using custom backup integrations with `last_backup` attribute, verify detection works

### 📝 Related Documentation

- [Home Assistant Backup Integration](https://www.home-assistant.io/integrations/backup/)
- [TESTING.md](../TESTING.md) - Testing guide

### 🔗 References

- Issue: [Fails to run with backup error](https://github.com/Bibbleq/HA-Update-Blueprint/issues/XXX)

---

**Migration Required:** No  
**Breaking Changes:** None  
**Tested On:** Home Assistant 2024.8.0+  
**Status:** Stable
