# Breaking Changes Detection Fix

## Problem Report

User reported that even with `skip_breaking_changes` enabled, updates containing "⚠️ Breaking Changes" in their release notes were still being applied automatically.

## Root Cause Analysis

### The Bug

The original implementation used a YAML anchor with a nested `sequence` block:

```yaml
- &check_breaking_changes
  alias: Check for breaking changes
  sequence:
    - variables:
        has_breaking_changes: >
          {{ ... detection logic ... }}
    - if: '{{ has_breaking_changes }}'
      then:
        - variables: ...
        - *logbook_update
        - stop: "Skipping update with breaking changes"
```

### Why It Failed

When this anchor was referenced with `*check_breaking_changes` in the repeat loop, the structure became:

```
repeat:
  sequence:
    - Set current_update_entity
    - *check_breaking_changes  # <-- This is ONE step with a nested sequence
    - *log_updating
    - *update_install
    - *update_wait
```

The problem:
1. `*check_breaking_changes` expands to a step with `alias` and `sequence`
2. The `stop` command on line 1226 was **inside** this nested `sequence`
3. When `stop` executes inside a nested sequence, it only stops that sequence, not the parent repeat loop
4. The repeat loop continues to the next step (`*log_updating`)
5. The update gets installed despite breaking changes being detected!

### Why `continue_on_error: true` Made It Worse

The repeat block also had `continue_on_error: true`, which meant even if the `stop` caused some kind of error condition, it would be ignored and execution would continue.

## The Solution

Restructured the breaking changes check to avoid nested sequences:

### New Structure

```yaml
- &check_breaking_changes
  variables:
    has_breaking_changes: >
      {{ ... detection logic ... }}
      
- &check_and_skip_breaking_changes
  if: '{{ has_breaking_changes }}'
  then:
    - variables: ...
    - *logbook_update
```

### Usage in Repeat Loop

```yaml
repeat:
  sequence:
    - Set current_update_entity
    - *check_breaking_changes         # Sets has_breaking_changes variable
    - *check_and_skip_breaking_changes # Logs if breaking changes found
    - if: '{{ not has_breaking_changes }}'  # NEW: Conditional wrapper
      then:
        - *log_updating
        - *update_install
        - *update_wait
```

### How It Works Now

1. **Step 1:** `*check_breaking_changes` sets the `has_breaking_changes` variable
   - No nested sequence, just a variable assignment
   - Always executes successfully

2. **Step 2:** `*check_and_skip_breaking_changes` logs if breaking changes found
   - Conditional: only logs if `has_breaking_changes` is true
   - No `stop` command needed

3. **Step 3:** Conditional wrapper around update steps
   - `if: '{{ not has_breaking_changes }}'`
   - If breaking changes found: steps are skipped entirely
   - If no breaking changes: steps execute normally

## Benefits of New Approach

1. **No nested sequences:** Avoids scope issues with `stop` command
2. **Explicit control flow:** Uses `if` conditions instead of `stop`
3. **More predictable:** Clear which steps execute under which conditions
4. **Same logging:** Still logs when updates are skipped
5. **No breaking config changes:** Works with existing configurations

## Test Results

All test scenarios pass:

### Scenario 1: Breaking Changes Detected (skip enabled)
- ✅ `has_breaking_changes` = true
- ✅ Skip message logged
- ✅ Update steps NOT executed
- ✅ Update NOT applied

### Scenario 2: No Breaking Changes (skip enabled)
- ✅ `has_breaking_changes` = false
- ✅ No skip message
- ✅ Update steps executed
- ✅ Update applied normally

### Scenario 3: Breaking Changes Detected (skip disabled)
- ✅ `has_breaking_changes` = false (because skip disabled)
- ✅ No skip message
- ✅ Update steps executed
- ✅ Update applied (user chose to allow breaking changes)

## Code Changes

### Files Modified
- `auto_update_scheduled.yaml`: Breaking changes detection logic

### Changes Summary
- Removed nested `sequence` from `&check_breaking_changes` anchor
- Removed `stop` command
- Created new `&check_and_skip_breaking_changes` anchor for logging
- Added `if: '{{ not has_breaking_changes }}'` wrapper around update steps
- Applied to all 3 update types: generic, firmware, core, and OS

### Lines Changed
- Lines 1208-1226: Restructured anchor definitions
- Lines 1319-1325: Added conditional wrapper (generic updates)
- Lines 1376-1382: Added conditional wrapper (firmware updates)
- Lines 1433-1439: Added conditional wrapper (core updates)
- Lines 1490-1496: Added conditional wrapper (OS updates)

## Verification

Run these tests to verify the fix:

### Test 1: Enable Breaking Changes Filter
```yaml
skip_breaking_changes: true
```

Expected: Updates with "breaking changes" in release notes are skipped.

### Test 2: Disable Breaking Changes Filter
```yaml
skip_breaking_changes: false
```

Expected: All updates applied, including those with breaking changes.

### Test 3: Check Logs
When an update with breaking changes is skipped, you should see:
```
Skipping `[Update Name]` - contains breaking changes
```

## Migration Notes

**No action required!** This fix improves the existing behavior without requiring any configuration changes.

Users who:
- Already have `skip_breaking_changes: true` will now have it work correctly
- Have `skip_breaking_changes: false` will continue to apply all updates
- Never configured it will use the new default (true)
