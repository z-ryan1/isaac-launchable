# Deploying a Trained Policy via Isaac ROS

In this lesson, we will explore how to deploy a trained reinforcement learning (RL) policy using Isaac ROS. Deploying RL policies in similationated environments is a crucial step towards enabling robots to perform complex tasks in real-world scenarios.

We will cover the necessary steps to integrate the trained policy into an Isaac ROS application, allowing the robot to perform tasks in a physical environment.

```{figure} ../images/sim-assembly.gif
```

## Workflow Configuration

The manipulator configuration file is available at `${ISAAC_ROS_WS}/isaac_manipulator_config/my_robot_config.yaml` and has been pre-configured for convenience.
Below are notable parameters for the gear assembly workflow:

```yaml
workflow_type: GEAR_ASSEMBLY

gear_assembly_model_path: $ISAAC_ROS_WS/isaac_sim_assets/policy/
gear_assembly_model_file_name: model.pt

use_ground_truth_pose_in_sim: 'true'
gear_assembly_use_ground_truth_pose_in_sim: 'true'

gear_assembly_offset_for_place_pose: 0.31
gear_assembly_offset_for_insertion_pose: 0.0
gear_assembly_timeout_for_insertion_action_call: 10.0
gear_assembly_model_frequency: 30.0
```

## Launch Isaac Sim

1. In a new terminal, set `ROS_DOMAIN_ID`:

    ```bash
    export ROS_DOMAIN_ID=<ID>
    ```

1. Start Isaac Sim with the following command:

    ```bash
    ${ISAACSIM}/isaac-sim.sh
    ```

    > **Note**: Wait for `Isaac Sim Full App is loaded.` before moving forward.

1. Open the Isaac Manipulator scene:
    1. Select the `Content` panel at the bottom of the UI.

    1. Copy and paste the following in navigation bar:
        ```bash
        /home/ubuntu/Downloads/isaac_manipulator_scene/isaac_manipulator_scene_fixed.usd
        ```
        Alternatively, use the content browser and navigate to `My Computer` > `Downloads` > `isaac_manipulator_scene` and right-click `isaac_manipulator_scene_fixed.usd` and select `Open`.

     1. Select `Don't Save` when asked to save the stage.

1. After the scene finishes loading, hit **Play** to start the simulation.

## Launch Gear Assembly

### Initialize the Gear Assembly Workflow

1. In a new terminal, set `ROS_DOMAIN_ID`:

    ```bash
    export ROS_DOMAIN_ID=<ID>
    ```

1. Source the ROS workspace:

    ```bash
    cd ${ISAAC_ROS_WS} && source install/setup.bash
    ```

1. Launch the gear assembly workflow:

    ```bash
    ros2 launch isaac_manipulator_bringup workflows.launch.py \
        manipulator_workflow_config:=${ISAAC_ROS_WS}/isaac_manipulator_config/my_robot_config.yaml
    ```

    > **Note**: Wait for `cuMotion is ready for planning queries!` before moving forward.

### Trigger Gear Assembly

1. In a new terminal, set `ROS_DOMAIN_ID`:

    ```bash
    export ROS_DOMAIN_ID=<ID>
    ```

1. Source the ROS workspace:

    ```bash
    cd ${ISAAC_ROS_WS} && source install/setup.bash
    ```

1. Trigger the gear assembly workflow:

    ```bash
    ros2 action send_goal /gear_assembly isaac_manipulator_interfaces/action/GearAssembly {}
    ```

1. The robot will pick up the gears and insert them into the peg stand in sequence.

1. The workflow should end after inserting the medium gear.
