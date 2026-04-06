# Setup

This course runs in an NVIDIA Brev environment with Isaac Lab pre-installed. You are already inside the environment if you are reading this in VS Code.

## Course files

All lesson markdown for this lab is under **`contact_rich_manipulation/docs/`** in the workspace (on the container, **`/workspace/contact_rich_manipulation/docs/`**).

## Opening a Terminal

Open a terminal in VS Code with `` Ctrl+` `` or via **Terminal → New Terminal** in the menu.

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

> **Tip:** Open the viewer tab first, then start the simulation. If the screen goes black, refresh the viewer tab after you see `Simulation App Startup Complete` in the terminal.

## Test the Stream

Before running the course, verify the viewer is working by launching an empty Isaac Sim scene:

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/tutorials/00_sim/create_empty.py --livestream 2 --enable_cameras
```

Open the viewer tab and confirm the Isaac Sim UI appears. Press `Ctrl+C` in the terminal to stop it when done.

## Next Steps

Continue to **`02-train.md`** to start training the gear assembly policy.
