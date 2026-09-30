# Windows Profile Cleaner

A lightweight desktop utility designed to manage and clean cached user profiles on shared computers, freeing drive space without damaging the Windows registry.

---

## Background & Problem

In shared workstation environments, multiple users sign in to the same machine over time. Each sign-in generates a local profile under `C:\Users`. Over time, this leads to:

- Storage exhaustion on the system drive (`C:`).
- Blocked system updates and degraded overall performance.

### Limitations of Manual Deletion
- **Time consuming:** Manually checking and removing each user directory requires significant manual effort.
- **Registry desynchronization:** Deleting profile folders directly from the filesystem leaves orphaned references in `HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`. This causes Windows to load subsequent sessions with temporary profiles (`TEMP`).

This project began as a PowerShell command-line script to address the registry desynchronization issue, and later evolved into a multi-threaded Python desktop application.

---

## Version History & Devlog

### Phase 1: Command-Line Automation (v0.1)

The initial implementation focused on removing profiles cleanly using Windows Management Instrumentation (WMI/CIM) through PowerShell. By referencing profile Security Identifiers (SIDs), the system removes directory data alongside the corresponding registry subkeys.

```powershell
Get-CimInstance Win32_UserProfile | Where-Object { -not $_.Special } | Remove-CimInstance
```

## Key Takeaways:

- Prevented temporary profile initialization errors.
- Automatically preserved default Windows system accounts using `-not $_.Special.`
- Required administrative command-line execution, which posed operational risks for general team use.

## Phase 2: Graphical Interface & Background Processing (v1.0)

To improve accessibility across the team, the cleanup logic was encapsulated in a Tkinter-based desktop interface.

Architecture & Concurrency:
Executing directory size calculations and `PowerShell` subprocesses on the main application thread caused the interface to become unresponsive. Background worker threads were implemented to preserve UI responsiveness:

```Python
def on_scan_clicked(self):
    threading.Thread(target=self.run_scan_thread, daemon=True).start()
```

A target quota mechanism was introduced to calculate target free space requirements, sort identified profiles by size descending, and remove only the profiles needed to satisfy the quota.

### Phase 3: Input Validation & Activity Logging (v1.5)
This update introduced stricter client-side validation and improved logging functionality.

Key Improvements:
- Keystroke validation: Prevented malformed user inputs on numeric criteria fields prior to execution:

```Python
def validate_float(self, user_input):
    if user_input == "" or user_input.count(".") <= 1:
        return user_input.replace(".", "", 1).isdigit() or user_input == ""
    return False
```
- Severity-tagged logging: Categorized storage impact visually within the log console (standard, moderate, and high capacity profiles).

- Drive audits: Implemented pre- and post-operation storage audits via `shutil.disk_usage`.

### Phase 4: Event-Driven Interruption & Dynamic Batch Filters (v2.0)
Version 2.0 refactored the operational loop to provide runtime control and automated profile discovery.

Key Additions:
- Controlled cancellation: Integrated `threading.Event()` into the recursive filesystem inspection routine to support safe, non-blocking cancellations:

```Python
for root_dir, dirs, files in os.walk(folder):
    if stop_event and stop_event.is_set():
        break
```
- Dynamic identifier grouping: Automatically inspects present user accounts and generates contextual filters (e.g., matching prefixes and older records).

- Post-execution synchronization: Refreshes profile tables automatically after task completion to mirror active disk state.

Technical Specifications
- User Interface: Python 3 (Tkinter / TTK)

- System Interface: PowerShell CIM instances `(Win32_UserProfile)`

- Concurrency: Python threading `(Thread, Event)`

- Filesystem Operations: `os.walk, shutil.disk_usage, subprocess`

Author
Omar Alzamel

Technical Support & Systems Automation
