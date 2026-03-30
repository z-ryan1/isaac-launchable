# Conclusion

In this course, you learned how to train and deploy a reinforcement learning policy (RL) for contact-rich robot manipulation using Isaac Lab and Isaac ROS.

## What You Accomplished

- **Trained a gear insertion RL policy** in Isaac Lab, defining observations, actions, and rewards tailored for precise contact-rich manipulation.
- **Validated the trained policy** in simulation using Isaac Lab's play mode and TensorBoard to confirm reliable gear insertion across varying conditions.
- **Deployed the policy via Isaac ROS**, bridging the gap from simulation to a robotics application stack capable of running on real hardware.

## Key Takeaways

- **Sim-to-real transfer requires careful alignment** between simulation and reality. Observations must use only real-world-available sensors, and the simulated robot's response to actions must match the physical robot's behavior.
- **Physics tuning is iterative.** Friction coefficients, contact solver parameters, and actuator models all need to be calibrated by comparing simulated and real-world behavior side by side.
- **Domain randomization builds robustness.** Randomizing actuator gains, joint friction, and object poses during training produces policies that generalize to the variability encountered in real-world deployment.
- **Action space design matters for contact-rich tasks.** Small, relative joint position commands give the policy fine-grained control needed for precise insertion without destabilizing contacts.

## Next Steps

To continue building on what you've learned:

- **Learn more about [Isaac Lab](https://isaac-sim.github.io/IsaacLab/main/index.html)**
    - **Try different manipulation tasks**: Apply the same workflow to other contact-rich tasks such as peg-in-hole, screw driving, or connector insertion.
    - **Experiment with observation and reward design**: Modify the observation space or reward shaping to see how it affects policy performance and transfer quality.
- **Explore the full capabilities of [Isaac ROS](https://nvidia-isaac-ros.github.io/)**
    - **Deploy on real hardware**: Use the Isaac ROS deployment pipeline to run your trained policies on a physical robot with percetion-in-the-loop.
