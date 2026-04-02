# Setup

This course runs in an NVIDIA Brev environment with Isaac Lab pre-installed. You are already inside the environment if you are reading this in VS Code.

## Opening the Isaac Sim Web Viewer

The Isaac Sim UI streams to a browser tab via Kit App Streaming. You can open it in two ways:

### In a separate tab
1. Run a command that starts Isaac Sim or Isaac Lab (see below)
2. Copy your current URL (e.g. `https://isaac.brevlab-1234`)
3. Open a new tab and change the end to `/viewer/` (e.g. `https://isaac.brevlab-1234/viewer`)
4. The Isaac Sim UI will appear when the app is ready. The page will say "Waiting for stream..." until then.

### Inside VS Code
1. Run a command that starts Isaac Sim or Isaac Lab (see below)
2. Press `Ctrl+Shift+P`
3. Type "Simple Browser: Show"
4. Enter URL: `/viewer/`
5. The Isaac Sim UI will appear when the app is ready.

## Opening a Terminal

Open a terminal in VS Code with `` Ctrl+` `` or via **Terminal → New Terminal** in the menu.

## Running Isaac Lab

You can run any Isaac Lab script with the streaming viewer using:

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/tutorials/00_sim/create_empty.py --livestream 2
```

For any other Isaac Lab command, append `--livestream 2` to stream the UI. Then open the Web Viewer to see it.

## VS Code Tasks

All key commands for this course are available as VS Code tasks — no need to type them manually:

1. Press `Ctrl+Shift+P`
2. Type "Tasks: Run Task"
3. Select from the "Contact Rich:" tasks listed
