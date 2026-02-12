# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project uses [Calendar Versioning](https://calver.org/) with the scheme `YYYY.MM.incremental`.

**Note:** Detailed technical changelogs for each version are available in the [`changelogs/`](changelogs/) directory. Exact release dates for versions v2025.10.3–v2025.10.11 were not preserved; these versions were released during October 2025–February 2026.

## [v2026.2.3] - 2026-02-12

### Fixed
- Fixed AI analysis receiving literal "None" text when `release_summary` and `release_notes` attributes are null
- Added multi-tier fallback for AI release notes prompt:
  1. Use inline notes when available
  2. Direct AI to fetch from `release_url` when populated (HACS, HA Core/OS)
  3. Provide firmware-specific guidance for device firmware updates (Z-Wave, Zigbee, etc.)
  4. Ask AI to search the web for add-ons and integrations with no notes or URL
- Removed condition requiring inline release notes to be present before calling AI agent
- Properly guard against `None` string values in release notes concatenation

## [v2026.2.2] - 2026-02-12

### Fixed
- Fixed `wait_template` for update completion never firing when a newer version is already available
- When multiple versions are queued (e.g., HACS integrations), the update entity stays `on` after install, causing the automation to hang for the full timeout duration
- Now detects completion by checking if `installed_version` changed, with fallback to state `off`
- Improved "Update - Remaining - Wait" batch wait to also detect version changes
- Enhanced success/error logging with version transition details (e.g., `v4.1.0 → v4.2.3`)

## [v2026.2.1] - 2026-02-12

### Changed
- **Versioning correction:** Adopted corrected Calendar Versioning (YYYY.MM.incremental). Previous versions (v2025.10.3–v2025.10.11) used a non-resetting incremental counter. Going forward, the incremental resets each calendar month.
- Moved documentation files to `docs/` directory for better repository organization:
  - `FIX_SUMMARY.md` → `docs/FIX_SUMMARY.md`
  - `REFACTORING_SUMMARY.md` → `docs/REFACTORING_SUMMARY.md`
  - `REFACTORING_COMPARISON.md` → `docs/REFACTORING_COMPARISON.md`
  - `IMPLEMENTATION_SUMMARY_v2025.10.7.md` → `docs/IMPLEMENTATION_SUMMARY_v2025.10.7.md`
  - `TESTING.md` → `docs/TESTING.md`
  - `VERIFICATION_CHECKLIST.md` → `docs/VERIFICATION_CHECKLIST.md`

### Added
- Created consolidated root `CHANGELOG.md` following Keep a Changelog format
- Updated version references throughout the repository to v2026.2.1
- Added link to CHANGELOG.md in README.md

## [v2025.10.11] - 2025-10-XX

### Changed
- Improved backup deduplication using timestamp checking instead of helper entity
- Backup creation logic now checks actual backup timestamps to prevent duplicates within 1 hour
- Helper entity now truly optional - only used for resume-after-restart detection
- Uses `sensor.backup_last_successful_automatic_backup` for timestamp checking

### Fixed
- More reliable backup management with better deduplication
- Reduced unnecessary storage usage from duplicate backups

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.11.md`](changelogs/CHANGELOG_v2025.10.11.md)

## [v2025.10.10] - 2025-10-XX

### Fixed
- Fixed backup age validation failing on modern Home Assistant with "No backup entity with last_backup attribute found" error
- Now uses `sensor.backup_last_successful_automatic_backup` state as primary check
- Falls back to legacy sensors with `last_backup` attribute for backwards compatibility
- Improved error message clarity when no backup entity is found

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.10.md`](changelogs/CHANGELOG_v2025.10.10.md)

## [v2025.10.9] - 2025-10-XX

### Added
- **AI-Powered Breaking Changes Analysis:** New feature using Home Assistant conversation integration
- Intelligent detection of breaking changes that may affect specific environment
- Supports any AI conversation agent (OpenAI, Google Generative AI, Anthropic, local LLMs, etc.)
- Customizable environment context for personalized breaking change detection
- Optional control to skip updates when AI flags concerns or just log insights
- AI query and response logged to System Log for review

### Changed
- Works alongside keyword-based detection for comprehensive protection

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.9.md`](changelogs/CHANGELOG_v2025.10.9.md)

## [v2025.10.8] - 2025-10-XX

### Changed
- **Overhauled backup monitoring:** Switched from polling to event-driven approach
- Changed backup service from `hassio.backup_full` to `backup.create_automatic`
- Added dual-trigger completion detection:
  - Primary: Monitors `sensor.backup_backup_manager_state` transition (create_backup → idle)
  - Fallback: Monitors `sensor.backup_last_successful_automatic_backup` updates
- Added backup duration tracking and detection method logging
- Removed backup location input (uses HA's configured backup location)

### Fixed
- Instant detection when backup completes (no polling delays)
- Eliminated false timeout warnings
- Improved compatibility with HA's native backup system

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.8.md`](changelogs/CHANGELOG_v2025.10.8.md)

## [v2025.10.7] - 2025-10-XX

### Fixed
- Fixed What-If mode still creating actual backups (now properly skips backup creation)
- Enhanced backup completion monitoring with fallback detection methods (idle state check)
- Added `[WHAT-IF MODE]` prefix to all notifications for better clarity
- Improved backup timeout messages with more detail

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.7.md`](changelogs/CHANGELOG_v2025.10.7.md)

## [v2025.10.6] - 2025-10-XX

### Fixed
- Fixed backups not being created when helper entity was configured
- Resolved timing issue with early helper entity state change

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.6.md`](changelogs/CHANGELOG_v2025.10.6.md)

## [v2025.10.5] - 2025-10-XX

### Fixed
- Fixed infinite loop in What-If mode that caused automation to repeat indefinitely
- Fixed updates still being installed despite What-If mode being enabled
- Added `processed_updates` tracking to normal mode success and timeout paths
- Wrapped "Update - Remaining" section in What-If mode conditional

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.5.md`](changelogs/CHANGELOG_v2025.10.5.md)

## [v2025.10.4] - 2025-10-XX

### Added
- **What-If Mode (dry run):** Safe feature for testing automation configuration
- Preview updates without making any changes
- All notifications work normally with "[WHAT-IF]" prefix
- No backups, updates, or restarts performed in What-If mode

### Documentation
- Added comprehensive What-If Mode documentation
- New usage example for What-If Mode testing

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.4.md`](changelogs/CHANGELOG_v2025.10.4.md)

## [v2025.10.3] - 2025-10-XX

### Fixed
- Fixed breaking changes detection not working correctly
- Fixed backup creation when helper entity not configured
- Corrected entity checking logic (treated as string instead of list)
- Fixed backup condition to work without helper entity
- Improved OR logic for backup creation

### Changed
- **BREAKING:** `skip_breaking_changes` now defaults to `true` (enabled) for safety
- **Migration Required:** Set `skip_breaking_changes: false` if you want all updates

**Detailed changelog:** [`changelogs/CHANGELOG_v2025.10.3.md`](changelogs/CHANGELOG_v2025.10.3.md)

---

[v2026.2.2]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2026.2.2
[v2026.2.1]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2026.2.1
[v2025.10.11]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.11
[v2025.10.10]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.10
[v2025.10.9]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.9
[v2025.10.8]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.8
[v2025.10.7]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.7
[v2025.10.6]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.6
[v2025.10.5]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.5
[v2025.10.4]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.4
[v2025.10.3]: https://github.com/Bibbleq/HA-Blueprint-Auto-Update-On-A-Schedule/releases/tag/v2025.10.3
