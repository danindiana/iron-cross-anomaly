# The "Iron Cross" UI Anomaly: A Linux Post-Mortem

## Overview
This repository documents a recurring UI hang and mouse cursor anomaly observed on Linux (X11) systems. The symptom involves the mouse cursor turning into an "iron cross" (crosshair) and terminal windows becoming unresponsive.

## The Root Cause: ImageMagick Collision & Systemd Persistence

The anomaly is caused by a **syntax collision** between Python script headers and the ImageMagick `import` utility, coupled with **Systemd User Services** that keep the failing scripts in a retry loop.

### Persistence Mechanism: The Systemd Loop
Even if the scripts are fixed manually, background processes like **Systemd Services** may be configured to execute these scripts in a retry loop. On this system, we identified both user-level and system-wide services (`/etc/systemd/system/agent-os-scheduler.service`, `ollama-proxy-admin.service`) that were triggering the anomaly every few seconds.

### The "Ghost" Process False Positive
During investigation, you may see strings containing "import" in the process list (e.g., in `ps aux`) that are **not** the ImageMagick utility. For example, the Rust compiler (`rustc`) uses flags like `--warn=clippy::wildcard_imports`. Always verify with `pgrep -x import` to ensure you are looking at the actual binary.

## Mitigation & Prevention

### Immediate Recovery
*   **Right-Click:** Cancel the ImageMagick selection mode.
*   **Esc Key:** Abort the capture.
*   **Kill Process:** `sudo killall -9 import`

### Permanent Prevention
1.  **Mandatory Shebangs:** Every Python script must start with `#!/usr/bin/env python3`.
2.  **Explicit Invocation:** Always run scripts using the interpreter: `python3 script.py`.
3.  **Shell Protection:** Add an alias to your `.bashrc` or `.zshrc` to block the binary:
    ```bash
    alias import='echo "Blocked ImageMagick import to prevent UI hang."'
    ```
4.  **Service Audit:** Disable any looping systemd services that reference un-shebanged scripts:
    ```bash
    sudo systemctl disable --now agent-os-scheduler.service
    ```

## Visualizations
See the `diagrams/` directory for Graphviz representations of the failure logic.
