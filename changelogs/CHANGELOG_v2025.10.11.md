# Changelog: v2025.10.11 - Improved Backup Deduplication

## Summary

Enhanced backup creation logic to use timestamp-based deduplication instead of relying solely on the helper entity. This prevents duplicate backups more reliably while keeping the helper entity optional for resume-after-restart detection.

## Changes

### Improved Backup Deduplication Logic

**What Changed:**
- Backup creation now checks `sensor.backup_last_successful_automatic_backup` timestamp
- Only creates a backup if the last one is older than 1 hour (3600 seconds)
- Respects backups created by ANY process, not just this automation
- Helper entity is now truly optional - only used for resume-after-restart detection

**Before:**
```yaml
# Only checked helper entity state
if:
  - '{{ input_backup_bool }}'
  - condition: or
    conditions:
      - '{{ input_update_process_started_entity | string | length == 0 }}'
      - condition: state
        entity_id: !input update_process_started_entity
        state: 'off'
```

**After:**
```yaml
# Checks resume state AND backup timestamp
if:
  - '{{ input_backup_bool }}'
  - '{{ not is_resume_after_restart }}'
  - condition: template
    value_template: >
      {% set backup_sensor = states('sensor.backup_last_successful_automatic_backup') %}
      {% if backup_sensor not in ['unknown', 'unavailable', ''] %}
        {{ (as_timestamp(now()) - as_timestamp(backup_sensor)) > 3600 }}
      {% else %}
        true
      {% endif %}
```

## Benefits

✅ **More Accurate**: Respects all backups, not just automation-created ones  
✅ **Configurable Threshold**: 1-hour window prevents duplicate backups  
✅ **Helper Entity Optional**: Works perfectly without a helper entity  
✅ **Resume After Restart**: Still uses helper entity for restart detection (when configured)  
✅ **Backward Compatible**: Existing configurations continue to work  

## Migration Guide

No action required! This is a drop-in improvement that works with existing configurations.

### Optional: Remove Helper Entity

If you were only using the helper entity for backup deduplication, you can now remove it:

1. Remove `update_process_started_entity` from your automation configuration
2. Delete the `input_boolean` helper entity if you created one
3. The automation will now use timestamp-based deduplication only

### Keep Helper Entity for Resume Detection

If you want resume-after-restart detection:
- Keep your existing `update_process_started_entity` configuration
- The helper will be used ONLY for detecting restarts
- Backup deduplication will use timestamps regardless

## Testing Recommendations

### Test Scenario 1: No Helper Entity
**Setup:**
- Remove `update_process_started_entity` from configuration
- Run automation twice within 1 hour
- Verify second run skips backup creation

### Test Scenario 2: With Helper Entity
**Setup:**
- Configure `update_process_started_entity`
- Run automation, let it start updates
- Manually restart Home Assistant during updates
- Verify automation resumes without creating another backup

### Test Scenario 3: Manual Backup Respected
**Setup:**
- Manually create a backup
- Run automation within 1 hour
- Verify automation skips backup creation (respects manual backup)

## Technical Details

### Files Modified
- `auto_update_scheduled.yaml` - Backup creation condition updated
- `README.md` - Documentation updated
- `changelogs/CHANGELOG_v2025.10.11.md` - This changelog

### Backup Deduplication Time Window
- Default: 3600 seconds (1 hour)
- Rationale: Balances safety with preventing redundant backups
- Future: Could be made configurable via input parameter

## Compatibility

- ✅ Home Assistant 2024.8.0 or newer
- ✅ Works with modern `sensor.backup_last_successful_automatic_backup`
- ✅ Gracefully handles missing sensor (creates backup if sensor unavailable)
- ✅ Backward compatible with existing helper entity configurations

---

**Version:** v2025.10.11  
**Date:** February 2026  
**Impact:** Low risk - backward compatible enhancement
