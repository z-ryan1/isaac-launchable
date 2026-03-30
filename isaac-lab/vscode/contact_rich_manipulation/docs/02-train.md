# Training a Gear Insertion RL Policy in Isaac Lab

In this lesson, we will train a reinforcement learning (RL) policy to perform a contact-rich manipulation task. This task requires precise control and interaction with the environment, making it an excellent example for demonstrating the capabilities of RL in robotic manipulation.

## Overview

This lesson is based on the official Isaac Lab gear assembly tutorial. For complete documentation, see: [Training a Gear Insertion Policy and ROS Deployment](https://isaac-sim.github.io/IsaacLab/main/source/policy_deployment/02_gear_assembly/gear_assembly_policy.html)

### Visualizing the Environment

![Untrained policy visualization](../images/untrained_viz.gif)

Before diving into the technical details, lets to visualize the task and the state of the policy at the start of training:

```bash
${ISAACLAB}/isaaclab.sh -p ${ISAACLAB}/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-v0 \
    --num_envs 4
```

**Visual indicators during training:**
- A 6-DOF UR10e manipulator with a Robotiq gripper attached
- Initially, the robot will move somewhat randomly as the policy explores
- The gear will be positioned at various distances and orientations relative to the target shaft (due to pose randomization)
- At each reset the env resets with a random gear(large, medium, small) being grasped 

**Example of a trained policy:**

After training is complete, a successful policy will consistently insert the gear onto the shaft with precise alignment and smooth control:

![Gear Insertion Validation](../images/isaaclab_gear_insertion_validation.gif)

## Reinforcement Learning Overview

Reinforcement learning (RL) refers to training agents that learn by trial and error. RL works on the idea that rewarding or punishing an agent for its behavior makes it more likely to repeat or forego that behavior in the future.

The main parts of RL are the **agent** and the **environment**. At every step of interaction, the agent observes the current state of the world and decides on an action to take. The environment changes when the agent acts on it, and provides a **reward signal**—a number that tells the agent how good or bad the current state is. The goal of the agent is to maximize its cumulative reward over time.

The policy is the agent's "brain"—a neural network that maps observations to actions. Through training, the policy learns which actions lead to higher cumulative rewards, gradually improving its ability to insert the gear successfully.

<div align="center">

**Figure: During Training.**

![Policy Learning Progress](../images/rl_loop.png)

</div>

<div align="center">

**Figure: During Inference.**

![Inputs, Outputs, and Training Loop](../images/inputs_outputs_training.png)

</div>

At the end of training, you have a **trained policy**—a neural network that takes the current observations (joint states, gear/shaft poses, etc.) and outputs the actions (e.g. joint targets or torques) that maximize cumulative reward. In other words, the policy has learned to **predict which actions to take** in each state so that the robot achieves the intended goal (e.g. inserting the gear onto the shaft). Once trained, this policy can be deployed in simulation or on a real robot to perform the task from real-world sensor inputs.

**Policy learning progress (during training):**

<div style="display:none">

![Step 0 video](../images/rl-video-step-0.mp4)
![Step 5000 video](../images/rl-video-step-5000.mp4)
![Step 30000 video](../images/rl-video-step-30000.mp4)
![Step 215000 video](../images/rl-video-step-215000.mp4)
![Step 235000 video](../images/rl-video-step-235000.mp4)

</div>

<div align="center">

**Step 0**

<div class="policy-progress-gif">
<video autoplay loop muted playsinline><source src="../_images/rl-video-step-0.mp4" type="video/mp4"></video>
</div>

**Step 5,000**

<div class="policy-progress-gif">
<video autoplay loop muted playsinline><source src="../_images/rl-video-step-5000.mp4" type="video/mp4"></video>
</div>

**Step 30,000**

<div class="policy-progress-gif">
<video autoplay loop muted playsinline><source src="../_images/rl-video-step-30000.mp4" type="video/mp4"></video>
</div>

**Step 215,000**

<div class="policy-progress-gif">
<video autoplay loop muted playsinline><source src="../_images/rl-video-step-215000.mp4" type="video/mp4"></video>
</div>

**Step 235,000**

<div class="policy-progress-gif">
<video autoplay loop muted playsinline><source src="../_images/rl-video-step-235000.mp4" type="video/mp4"></video>
</div>

</div>

## Defining the RL Problem

To train a reinforcement learning policy for gear assembly, we need to define three key components: **observations** (what the robot can sense), **actions** (what the robot can do), and **rewards** (what we want the robot to achieve).

### Observations/Inputs

The observations define the state information available to the policy. For successful sim-to-real transfer, observations must only include information available from real sensors.

The gear assembly environment uses both **proprioceptive** (robot's internal state) and **exteroceptive** (perception from external sensors) observations:

| Observation | Dimension | Real-World Source | Sim Source |
|------------|-----------|-------------------|------------|
| `joint_pos` | 6 (UR10e arm joints) | UR10e controller | Ground truth articulation state |
| `joint_vel` | 6 (UR10e arm joints) | UR10e controller | Ground truth articulation state |
| `gear_shaft_pos` | 3 (x, y, z position) | FoundationPose + RealSense depth | Ground truth rigid body pose |
| `gear_shaft_quat` | 4 (quaternion orientation) | FoundationPose + RealSense depth | Ground truth rigid body pose |

**Implementation:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/gear_assembly_env_cfg.py
"""

@configclass
class ObservationsCfg:
    """Observation specifications for the MDP."""

    @configclass
    class PolicyCfg(ObsGroup):
        """Observations for policy group."""

        # Robot joint states - NO noise for proprioceptive observations
        joint_pos = ObsTerm(
            func=mdp.joint_pos,
            params={"asset_cfg": SceneEntityCfg("robot", joint_names=[".*"])}
        )

        joint_vel = ObsTerm(
            func=mdp.joint_vel,
            params={"asset_cfg": SceneEntityCfg("robot", joint_names=[".*"])}
        )

        # Gear shaft pose
        gear_shaft_quat = ObsTerm(func=mdp.gear_shaft_pos_w)

        gear_shaft_quat = ObsTerm(func=mdp.gear_shaft_quat_w)
```

### Actions/Outputs

The action space defines how the policy controls the robot. For contact-rich manipulation tasks, we use **relative joint position control** with a small action scale for precise movements.

**Implementation:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/config/ur_10e/joint_pos_env_cfg.py
"""

# Action scale for contact-rich manipulation (smaller = more precise)
self.joint_action_scale = 0.025  # ±2.5 degrees per step

self.actions.arm_action = mdp.RelativeJointPositionActionCfg(
    asset_name="robot",
    joint_names=[
        "shoulder_pan_joint",
        "shoulder_lift_joint",
        "elbow_joint",
        "wrist_1_joint",
        "wrist_2_joint",
        "wrist_3_joint",
    ],
    scale=self.joint_action_scale,
    use_zero_offset=True,
)
```

**Key Design Choices:**

- **Relative joint positions**: Actions are incremental changes to joint positions, not absolute targets. This provides smoother control.
- **Small action scale (0.025)**: For contact-rich tasks requiring precise control, we use ±2.5 degrees per control step. This prevents large, jerky movements that could cause the gear to slip.
- **use_zero_offset=True**: Actions are offsets from the current position, making the policy's actions relative to its current state.

### Rewards - What We Want the Robot to Achieve

The reward function guides the policy to learn the desired behavior. For gear insertion, we want the gear to align with and insert into the shaft.

**Implementation:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/gear_assembly_env_cfg.py
"""

@configclass
class RewardsCfg:
    """Reward terms for the MDP."""

    # Keypoint tracking reward (linear penalty for distance)
    end_effector_gear_keypoint_tracking = RewTerm(
        func=mdp.keypoint_entity_error,
        weight=-1.5,
        params={
            "asset_cfg_1": SceneEntityCfg("factory_gear_base"),
            "keypoint_scale": 0.15,
        },
    )

    # Keypoint tracking reward (exponential reward for proximity)
    end_effector_gear_keypoint_tracking_exp = RewTerm(
        func=mdp.keypoint_entity_error_exp,
        weight=1.5,
        params={
            "asset_cfg_1": SceneEntityCfg("factory_gear_base"),
            "kp_exp_coeffs": [(50, 0.0001), (300, 0.0001)],
            "kp_use_sum_of_exps": False,
            "keypoint_scale": 0.15,
        },
    )

    # Action rate penalty (discourage large, jerky movements)
    action_rate = RewTerm(func=mdp.action_rate_l2, weight=-5.0e-06)
```

![Reward Function Visualization](../images/rewards.png)

*Visualization of the pose error calculation guiding the policy to align and insert the gear.*

**Key Design Choices:**

- **Keypoint tracking**: Measures the distance between the gear and the target shaft position. Uses two terms:
  - **Linear penalty** (`weight=-1.5`): Provides a constant gradient pointing toward the goal
  - **Exponential reward** (`weight=1.5`): Provides stronger signal when very close to the goal, encouraging precise alignment
- **Action rate penalty** (`weight=-5.0e-06`): Penalizes large changes in actions between timesteps, encouraging smooth movements
- **keypoint_scale=0.15**: Defines the characteristic distance (15cm) for the reward shaping

![Episode Reward and Reward Components Over Training](../images/rewards_graph.png)

*Episode reward and individual reward terms (e.g. action rate, end-effector–gear distance) over training steps. The policy improves rapidly at first, then converges to high reward.*

## Physics Simulation and Domain Randomization

After defining what the robot senses (observations), what it can do (actions), and what we want it to achieve (rewards), we need to ensure the simulation behaves like the real world. This involves configuring accurate physics simulation and applying domain randomization to bridge the sim-to-real gap.

This section covers:
1. **Physics parameters**: Contact properties, friction, solver settings
2. **Actuator modeling**: PD gains, effort limits, joint friction
3. **Domain randomization**: Randomizing physics properties to improve robustness

### Friction and Material Properties

Accurate physics simulation is critical for contact-rich tasks. The gear assembly environment configures friction coefficients that match real-world behavior.

**Implementation:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/config/ur_10e/joint_pos_env_cfg.py
"""

@configclass
class EventCfg:
    """Configuration for events including physics randomization."""

    # Randomize friction for gear objects
    small_gear_physics_material = EventTerm(
        func=mdp.randomize_rigid_body_material,
        mode="startup",
        params={
            "asset_cfg": SceneEntityCfg("factory_gear_small", body_names=".*"),
            "static_friction_range": (0.75, 0.75),
            "dynamic_friction_range": (0.75, 0.75),
            "restitution_range": (0.0, 0.0),
            "num_buckets": 16,
        },
    )

    # Similar configuration for gripper fingers
    robot_physics_material = EventTerm(
        func=mdp.randomize_rigid_body_material,
        mode="startup",
        params={
            "asset_cfg": SceneEntityCfg("robot", body_names=".*finger"),
            "static_friction_range": (0.75, 0.75),
            "dynamic_friction_range": (0.75, 0.75),
            "restitution_range": (0.0, 0.0),
            "num_buckets": 16,
        },
    )
```

These friction values (0.75) were determined through iterative visual comparison:

1. Record videos of the gear being grasped and manipulated on real hardware
2. Start training in simulation and observe the live simulation viewer
3. Look for physics issues (penetration, unrealistic slipping, poor contact)
4. Adjust friction coefficients and solver parameters and retry
5. Compare the gear's behavior in the gripper visually between sim and real
6. Repeat adjustments until behavior matches

### Contact Solver Configuration

Contact-rich manipulation requires careful solver tuning to handle the complex interactions between the gear and gripper:

```python
# Robot rigid body properties
rigid_props=sim_utils.RigidBodyPropertiesCfg(
    disable_gravity=True,
    max_depenetration_velocity=5.0,
    linear_damping=0.0,
    angular_damping=0.0,
    max_linear_velocity=1000.0,
    max_angular_velocity=3666.0,
    enable_gyroscopic_forces=True,
    solver_position_iteration_count=4,  # Balance accuracy vs performance
    solver_velocity_iteration_count=1,
    max_contact_impulse=1e32,
),

# Articulation properties
articulation_props=sim_utils.ArticulationRootPropertiesCfg(
    enabled_self_collisions=False,
    solver_position_iteration_count=4,
    solver_velocity_iteration_count=1,
),

# Contact properties
collision_props=sim_utils.CollisionPropertiesCfg(
    contact_offset=0.005,  # 5mm contact detection distance
    rest_offset=0.0,
),
```

**Important**: The `solver_position_iteration_count` is a critical parameter for contact-rich tasks. Increasing this value improves collision simulation stability and reduces penetration issues, but it also increases simulation and training time. For the gear assembly task, we use `solver_position_iteration_count=4` as a balance between physics accuracy and computational performance.

### Actuator Modeling

Accurate actuator modeling ensures the simulated robot moves like the real one. For the UR10e, we model the impedance controller with calibrated stiffness and damping values.

**Implementation:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/config/ur_10e/joint_pos_env_cfg.py
"""

actuators = {
    "arm": ImplicitActuatorCfg(
        joint_names_expr=["shoulder_pan_joint", "shoulder_lift_joint",
                        "elbow_joint", "wrist_1_joint", "wrist_2_joint", "wrist_3_joint"],
        effort_limit=87.0,    # From UR10e specifications
        velocity_limit=2.0,   # From UR10e specifications
        stiffness=800.0,      # Calibrated to match real behavior
        damping=40.0,         # Calibrated to match real behavior
    ),
}
```

**Domain Randomization of Actuator Parameters**

To account for variations in real robot behavior, randomize actuator gains during training:

```python
robot_joint_stiffness_and_damping = EventTerm(
    func=mdp.randomize_actuator_gains,
    mode="reset",
    params={
        "asset_cfg": SceneEntityCfg("robot", joint_names=["shoulder_.*", "elbow_.*", "wrist_.*"]),
        "stiffness_distribution_params": (0.75, 1.5),  # 75% to 150% of nominal
        "damping_distribution_params": (0.3, 3.0),     # 30% to 300% of nominal
        "operation": "scale",
        "distribution": "log_uniform",
    },
)
```

**Joint Friction Randomization**

Real robots have friction in their joints that varies with position, velocity, and temperature. For the UR10e with impedance controller interface, we observed significant stiction (static friction) causing the controller to not reach target joint positions.

```python
joint_friction = EventTerm(
    func=mdp.randomize_joint_parameters,
    mode="reset",
    params={
        "asset_cfg": SceneEntityCfg("robot", joint_names=["shoulder_.*", "elbow_.*", "wrist_.*"]),
        "friction_distribution_params": (0.3, 0.7),  # Add 0.3 to 0.7 Nm friction
        "operation": "add",
        "distribution": "uniform",
    },
)
```

**Why Joint Friction Matters**: Without modeling joint friction in simulation, the policy learns to expect that commanded joint positions are always reached. On the real robot, stiction prevents small movements and causes steady-state errors. By adding friction during training, the policy learns to account for these effects.

### Environment Pose Randomization

To ensure the policy generalizes to different gear and base positions, we randomize their poses at each episode reset. This trains the policy to handle variations in object placement that occur in real-world deployment.

**Gear and Base Pose Randomization:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/config/ur_10e/joint_pos_env_cfg.py
"""

randomize_gears_and_base_pose = EventTerm(
    func=gear_assembly_events.randomize_gears_and_base_pose,
    mode="reset",
    params={
        "pose_range": {
            "x": [-0.1, 0.1],                    # ±10cm
            "y": [-0.25, 0.25],                  # ±25cm
            "z": [-0.1, 0.1],                    # ±10cm
            "roll": [-math.pi/90, math.pi/90],   # ±2 degrees
            "pitch": [-math.pi/90, math.pi/90],
            "yaw": [-math.pi/6, math.pi/6],      # ±30 degrees
        },
        "gear_pos_range": {
            "x": [-0.02, 0.02],                  # ±2cm relative to base
            "y": [-0.02, 0.02],
            "z": [0.0575, 0.0775],               # 5.75-7.75cm above base
        },
        "rot_randomization_range": {
            "roll": [-math.pi/36, math.pi/36],   # ±5 degrees
            "pitch": [-math.pi/36, math.pi/36],
            "yaw": [-math.pi/36, math.pi/36],
        },
    },
)
```

**Robot Initial State Randomization:**

```python
"""
${ISAACLAB}/source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/deploy/gear_assembly/config/ur_10e/joint_pos_env_cfg.py
"""

set_robot_to_grasp_pose = EventTerm(
    func=gear_assembly_events.set_robot_to_grasp_pose,
    mode="reset",
    params={
        "robot_asset_cfg": SceneEntityCfg("robot"),
        "rot_offset": [0.0, math.sqrt(2)/2, math.sqrt(2)/2, 0.0],
        "pos_randomization_range": {
            "x": [-0.0, 0.0],
            "y": [-0.005, 0.005],  # ±5mm variation in y-axis
            "z": [-0.003, 0.003],  # ±3mm variation in z-axis
        },
        "gripper_type": "2f_140",
    },
)
```

This initializes the robot to a pre-grasp pose with small variations, simulating slight differences in gripper positioning when grasping the gear.

## Train RL Policy

### Visualize the Environment

First, launch the training with a small number of environments and visualization enabled to verify that the environment is set up correctly:

```bash
${ISAACLAB}/isaaclab.sh -p ${ISAACLAB}/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --num_envs 4
```

This will open the Isaac Sim viewer where you can observe the training process in real-time.

**What to Expect**: In the early stages of training, you should see the robots moving around with the gears grasped by the grippers, but they won't be successfully inserting the gears yet. This is expected behavior as the policy is still learning. Once you've verified the environment looks correct, stop the training (Ctrl+C) and proceed to full-scale training.

### Full-Scale Training with Video Recording

Now launch the full training run with more parallel environments in headless mode for faster training. We'll also enable video recording to monitor progress:

> **Note**: Do not run live in class.

```bash
${ISAACLAB}/isaaclab.sh -p ${ISAACLAB}/scripts/reinforcement_learning/rsl_rl/train.py \
    --task Isaac-Deploy-GearAssembly-UR10e-2F140-ROS-Inference-v0 \
    --headless \
    --num_envs 256 \
    --video --video_length 800 --video_interval 5000
```

This command will:

- Run 256 parallel environments for efficient training
- Run in headless mode (no visualization) for maximum performance
- Record videos every 5000 steps to monitor training progress
- Save videos with 800 frames each

Training typically takes ~12-24 hours for a robust insertion policy. The videos will be saved in the logs directory and can be reviewed to assess policy performance during training.

> **Note**: GPU Memory Considerations: The default configuration uses 256 parallel environments, which should work on most modern GPUs (e.g., RTX 3090, RTX 4090, A100). For better sim-to-real transfer performance, you can increase `solver_position_iteration_count` from 4 to 196 for more realistic contact simulation, but this requires a larger GPU (e.g., RTX PRO 6000 with 40GB+ VRAM).

## Sim-to-Real Transfer

The policy is numbers in → numbers out. For it to work on the real robot:

1. **Observations** in sim and real must match (same quantities, units, order).
2. **Output response** on the real robot must match the sim: the robot must respond to predicted actions the same way (actuation, latency).
