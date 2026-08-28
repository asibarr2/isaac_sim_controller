# isaac_sim_controller

A ROS Python script for publishing velocity commands to a robot's `/cmd_vel` topic, adapted for controlling a robot (e.g. Carter/JetBot) inside NVIDIA Isaac Sim.

## Overview

This is based on ROS's standard `teleop_twist_keyboard.py`, modified to publish `geometry_msgs/Twist` messages on the `cmd_vel` topic via a dedicated publisher thread. The publisher runs continuously in the background, sending linear and angular velocity commands to drive the robot.

## How it works

- `PublishThread` runs as a background thread, publishing `Twist` messages to `cmd_vel` at a configurable rate.
- `wait_for_subscribers()` blocks until something (e.g. the simulated robot in Isaac Sim) is actually subscribed to `cmd_vel`, so commands aren't published into the void before the robot is ready.
- `update()` lets the main thread push new velocity values into the publisher thread in a thread-safe way (using a condition variable).
- On shutdown, `stop()` sends a final zero-velocity `Twist` so the robot comes to rest cleanly rather than continuing on its last command.

## Requirements

- ROS (developed against the `teleop_twist_keyboard` package's manifest/dependencies)
- A subscriber listening on `cmd_vel` — in this context, the Isaac Sim robot/environment (e.g. from the companion [Omni-RL-Isaac](https://github.com/asibarr2/Omni-RL-Isaac) or [City-Park-USD](https://github.com/asibarr2/City-Park-USD) setup)

## Usage

```bash
rosrun <your_package> test.py
```

ROS parameters (set via `rosparam` or launch file):
- `~speed` (default `0.5`) — linear speed scale
- `~turn` (default `1.0`) — angular turn scale
- `~repeat_rate` (default `0.0`) — publish rate in Hz; `0.0` means only publish on new input rather than continuously
- `~key_timeout` (default `0.0`) — timeout for keyboard input polling

## Notes / current state

- This version currently publishes a **fixed** velocity command (`linear.x = 0.5`, `angular.z = 0.5`) inside the publish loop rather than reading live keyboard input — the interactive key-reading logic present in the original `teleop_twist_keyboard.py` (arrow-key/WASD-style controls) has been stripped out in this version. As-is, it drives the robot forward while turning at a constant rate rather than responding to real-time input.
- To restore interactive keyboard control, the `getKey()`-based input loop from the original ROS `teleop_twist_keyboard.py` would need to be reintroduced in `__main__`, feeding live key presses into `pub_thread.update(...)`.
