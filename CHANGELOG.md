# Changelog

All notable changes to the Windows Profile Cleaner project will be documented in this file.

## [2.0.0] - 2026-09-30
### Added
- Graceful task cancellation using `threading.Event()` inside recursive file sweeps.
- Dynamic profile prefix discovery with "Batch & Older" automated grouping.
- Post-execution synchronization to refresh profile status automatically.

## [1.5.0] - 2026-09-15
### Added
- Real-time keystroke input validation (`validate="key"`) for integer and float fields.
- Multi-tier color-coded console logs (Green, Orange, Red) based on profile capacity.
- Pre- and post-operation storage audits via `shutil.disk_usage`.

## [1.0.0] - 2026-09-01
### Added
- Initial Tkinter desktop graphical user interface.
- Asynchronous worker threading to decouple storage I/O from the GUI main loop.
- Target quota cleanup automation.

## [0.1.0] - 2026-08-15
### Added
- Initial PowerShell CLI implementation using `Get-CimInstance Win32_UserProfile`.
- Clean deletion targeting account SIDs to prevent registry desynchronization.
