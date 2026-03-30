# Isaac Launchable Usage

This project provides a simple way to interact with Isaac Lab and Isaac Sim, either on the cloud or locally.
See the [project repo](https://github.com/isaac-sim/isaac-launchable) for more detailed instructions, but below is a quickstart guide.

## Web Viewer for Isaac Sim UI

### To view in a separate tab
1. Run a command that starts Isaac Sim, or Isaac Lab (see below)
2. Copy your current URL (ie `https://isaac.brevlab-1234`)
3. Open a new tab in your browser and paste the URL, changing the end to `/viewer/` (ie `https://isaac.brevlab-1234/viewer`)
4. The Isaac Sim UI will appear when the app is ready. The page will say "Waiting for stream..." until then.

### To view inside VSCode
1. Run a command that starts Isaac Sim, or Isaac Lab (see below)
2. Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac)
3. Type "Simple Browser: Show"
4. Enter URL: `/viewer/`
5. The Isaac Sim UI will appear when the app is ready. The page will say "Waiting for stream..." until then.

## Running Isaac Lab 2.3

You can run any of the Isaac Lab scripts with the streaming Isaac Sim experience with the following command:

```console
cd /workspace/isaaclab
./isaaclab.sh -p scripts/tutorials/00_sim/create_empty.py --livestream 2
```

To run any other Isaac Lab commands, simply append the same argument as shown above: `--livestream 2`.

Then follow the Web Viewer for Isaac Sim UI instructions, if not using headless mode.

## Contact Rich Robot Manipulation (GTC26)

This launchable includes the GTC26 course materials for "Train and Deploy Contact-Rich Robot Manipulation Skills."
The course covers training a UR10e + Robotiq 2F140 gripper to insert gears using reinforcement learning in Isaac Lab.

Course docs are at `/workspace/contact_rich_manipulation/docs/` — open any `.md` file in VS Code to read them.

### Visualize the gear assembly environment

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-v0 \
    --num_envs 4 --livestream 2
```

Then open the Web Viewer to observe the simulation.

### Train the policy (headless, full scale)

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --headless --num_envs 256 \
    --video --video_length 800 --video_interval 5000
```

Training typically takes 12-24 hours. Logs are saved under `/workspace/logs/`.

### Validate a trained policy

```console
$ISAACLAB/isaaclab.sh -p $ISAACLAB/scripts/reinforcement_learning/rsl_rl/play.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --num_envs 4 --checkpoint /path/to/model.pt --livestream 2
```

### Monitor training with TensorBoard

```console
$ISAACLAB/isaaclab.sh -p -m tensorboard.main --logdir /workspace/logs/rsl_rl/gear_assembly
```

Then open `http://localhost:6006` in your browser.

All of the above commands are also available as VS Code tasks (`Ctrl+Shift+P` → "Tasks: Run Task").

## Running Isaac Sim 5.1

You can run the streaming Isaac Sim application at any time with the following command.

```console
# Option 1: requires EULA acceptance
/isaac-sim/runheadless.sh

# Option 2: to automatically accept the EULA:
ACCEPT_EULA=y /isaac-sim/runheadless.sh
```

Then follow the Web Viewer for Isaac Sim UI instructions above, if not using headless mode.
