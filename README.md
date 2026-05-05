# The "Iron Cross" UI Anomaly: A Linux Post-Mortem

## Overview
This repository documents a recurring UI hang and mouse cursor anomaly observed on Linux (X11) systems. The symptom involves the mouse cursor turning into an "iron cross" (crosshair) and terminal windows becoming unresponsive.

## The Root Cause: ImageMagick Collision
The anomaly is caused by a **syntax collision** between Python script headers and the ImageMagick `import` utility.

### Failure Chain
1.  **Execution:** A Python script (e.g., `script.py`) is executed as a shell script (e.g., via `./script.py` or an automated agent).
2.  **Missing Shebang:** The script lacks the mandatory `#!/usr/bin/env python3` header.
3.  **Shell Interpretation:** The Linux shell (bash/zsh) attempts to execute the script line-by-line.
4.  **The Collision:** The first line of many Python scripts is `import os` or `import sys`.
5.  **Utility Launch:** The shell sees the `import` command and, instead of treating it as a Python keyword, launches the **ImageMagick `import` utility**.
6.  **UI Capture:** The ImageMagick `import` tool is a screenshot utility that waits for a window selection. It:
    *   Changes the cursor to the "iron cross" (`XC_X_cursor`).
    *   Captures the X11 mouse/keyboard focus.
    *   Freezes terminal refreshing until a selection is made or the process is killed.

## Identification
To confirm this issue is happening, run:
```bash
ps aux | grep "import"
```
If you see a process like `import unicodedata` or `import os`, the collision has occurred.

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
