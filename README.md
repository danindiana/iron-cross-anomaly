# The "Iron Cross" UI Anomaly: A Linux Post-Mortem

## Overview
This repository documents a recurring UI hang and mouse cursor anomaly observed on Linux (X11) systems. The symptom involves the mouse cursor turning into an "iron cross" (crosshair) and terminal windows becoming unresponsive.

## The Root Cause: ImageMagick Collision & Systemd Persistence

The anomaly is caused by a **syntax collision** between Python script headers and the ImageMagick `import` utility, coupled with **Systemd User Services** that keep the failing scripts in a retry loop.

### Persistence Mechanism
Even if the scripts are fixed manually, background processes like **Systemd User Services** (`agent-os-observer.service`, `agent-os-watchdog.service`) may be configured to execute these scripts frequently. If the underlying script lacks a shebang, the service will trigger the "iron cross" on every execution attempt.

### Identification
To confirm this issue is happening, run:
```bash
ps aux | grep "import"
systemctl --user list-units | grep "agent-os"
```
If you see a process like `import unicodedata` or `import os`, and systemd services are in a "failed" or "activating" state, the collision is active.


## Mitigation & Prevention

### Immediate Recovery
*   **Right-Click:** Cancel the ImageMagick selection mode.
*   **Esc Key:** Abort the capture.
*   **Kill Process:** `killall -9 import`

### Permanent Prevention
1.  **Mandatory Shebangs:** Every Python script must start with `#!/usr/bin/env python3`.
2.  **Explicit Invocation:** Always run scripts using the interpreter: `python3 script.py`.
3.  **Binary Aliasing (Optional):** Alias the `import` command in `~/.bashrc` to prevent accidental shell execution.

## Visualizations
See the `diagrams/` directory for Graphviz representations of the failure logic.
