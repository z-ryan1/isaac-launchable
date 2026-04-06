# Validating an RL Policy in Isaac Lab

In this lesson, we will validate a reinforcement learning (RL) policy that has been trained using IsaacLab.

## Visualize Training Metrics with TensorBoard

You can monitor training progress in real-time using TensorBoard.

In a terminal, run:

```bash
${ISAACLAB}/isaaclab.sh -p -m tensorboard.main --logdir /workspace/policy --bind_all
```

This points to the pre-trained policy logs bundled in this environment. If you have run your own training, replace `/workspace/policy` with your run’s log directory — for the gear-assembly tasks this will be under `/workspace/logs/rsl_rl/gear_assembly_ur10e/`; training prints `[INFO] Logging experiment in directory:` with the exact path. On Brev, open port **6006** in your instance settings to access TensorBoard in a browser.

Once running, open `http://<your-brev-instance-ip>:6006` in a browser to see the training metrics.

![Trained policy TensorBoard](images/trained_tensorboard.png)

## Validate Policy Performance in Isaac Lab

The policy performance can be validated by deploying it in Isaac Lab by running following command.

A pre-trained policy is included in this environment at `/workspace/policy/model.pt`. You can also point to a checkpoint from your own training run (RSL-RL saves `model_<iteration>.pt` inside a timestamped subdirectory under `/workspace/logs/rsl_rl/gear_assembly_ur10e/`).

```bash
${ISAACLAB}/isaaclab.sh -p ${ISAACLAB}/scripts/reinforcement_learning/rsl_rl/play.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --num_envs 4 \
    --checkpoint /workspace/policy/model.pt \
    --livestream 2
```

**`--livestream 2`** streams the viewport on this launchable (omit only if you use a local desktop window).

A successfully trained policy should be able to insert all 3 gears with the gear base in different locations.

![Gear insertion policy validation in Isaac Lab](images/isaaclab_gear_insertion_validation.gif)

You can stop the simulation by pressing `Ctrl+C` in the terminal.
