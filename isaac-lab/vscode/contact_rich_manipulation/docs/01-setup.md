# Setup

This course runs in an NVIDIA Brev environment with Isaac Lab pre-installed. You are already inside the environment if you are reading this in VS Code.

## Course files

All lesson markdown for this lab is under **`contact_rich_manipulation/docs/`** in the workspace (on the container, **`/workspace/contact_rich_manipulation/docs/`**).

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

## Quick command reference (gear assembly)

Details and context are in **`02-train.md`** and **`03-validate.md`**.

**Visualize the environment (few envs, livestream):**

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-v0 \
    --num_envs 4 --livestream 2
```

**Full-scale training (headless, many envs, videos):**

```console
cd /workspace/isaaclab
./isaaclab.sh -p $ISAACLAB/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --headless --num_envs 256 \
    --video --video_length 800 --video_interval 5000
```

Training often takes on the order of **12–24 hours**. RSL-RL logs under **`${ISAACLAB}/logs/rsl_rl/<experiment_name>/<run_timestamp>/`** when you launch from the Isaac Lab tree; for these gear tasks, **`experiment_name`** is **`gear_assembly_ur10e`**. The terminal prints **`[INFO] Logging experiment in directory:`** with the full path.

**TensorBoard** (after logs exist):

```console
$ISAACLAB/isaaclab.sh -p -m tensorboard.main --logdir /workspace/logs/rsl_rl/gear_assembly_ur10e --bind_all
```

On **Brev**, use port forwarding for **6006** (see **`03-validate.md`**) instead of assuming **`http://localhost:6006`** on your laptop reaches the instance.

**Validate with a checkpoint:**

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/reinforcement_learning/rsl_rl/play.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --num_envs 4 --checkpoint /workspace/policy/model.pt --livestream 2
```

Replace **`/path/to/model.pt`** with a **`model_<iteration>.pt`** inside the dated run folder under **`$ISAACLAB/logs/rsl_rl/gear_assembly_ur10e/`** (see **`03-validate.md`**).

After starting a livestreamed command, use the Web Viewer as described above.
