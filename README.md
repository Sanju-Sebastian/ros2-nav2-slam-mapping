# ROS2 Nav2 + SLAM Toolbox — Autonomous Mapping & Navigation

A hands-on project demonstrating the ROS2 Nav2 navigation stack combined with SLAM Toolbox for live map-building, using TurtleBot3 in Gazebo Harmonic simulation. This project focused on operating, diagnosing, and correctly sequencing the standard Nav2/SLAM pipeline rather than building custom nodes from scratch.

## What this demonstrates
- Launching and correctly sequencing the full Nav2 stack (`nav2_bringup`) with SLAM Toolbox enabled instead of a pre-built map
- Building an occupancy grid map from scratch via teleop-driven exploration, verified live in RViz
- Diagnosing and resolving real AMCL/costmap/planner failures encountered during testing (see below)
- Sending autonomous navigation goals on the robot's own built map (not a stock map), confirmed via successful goal completion in Nav2 logs

## What this is *not*
No custom ROS2 package, launch files, or parameter files were written for this project — it uses the stock `ros-jazzy-desktop` / `nav2_bringup` / `slam_toolbox` packages directly, with launch-time argument overrides (e.g. `slam:=True`). The value demonstrated here is correct operation and debugging of the stack, not custom stack architecture. For a project with fully custom, manually-wired Nav2 launch files and parameters, see the ROS2 Nav2 Manual Navigation Stack repository.

## Environment
- ROS2 Jazzy, Ubuntu 24.04, Gazebo Harmonic
- WSL2 with GPU/GUI passthrough (WSLg)
- TurtleBot3 Waffle

## How it was run

```bash
# SLAM mapping (fresh map, teleop-driven)
ros2 launch nav2_bringup tb3_simulation_launch.py slam:=True headless:=False

# Drive to build the map
ros2 run teleop_twist_keyboard teleop_twist_keyboard

# Save the completed map
ros2 run nav2_map_server map_saver_cli -f ~/amr_slam_map

# Reload navigation using the map just built
ros2 launch nav2_bringup tb3_simulation_launch.py map:=/home/sanju/amr_slam_map.yaml headless:=False
```

## Issues hit and resolved
- **Tool confusion mid-run**: repeatedly re-clicking RViz's "2D Pose Estimate" tool during active navigation (instead of only at startup) reset AMCL's belief mid-goal, breaking active path execution. Fixed by setting pose exactly once at launch and using "Nav2 Goal" exclusively afterward.
- **Planning failures on first goal attempts**: `GridBased plugin failed to plan... Failed to create plan with tolerance of: 0.500000` — caused by selecting goal points inside occupied/unreachable map cells. Fixed by visually confirming free space (white, not gray/black) in RViz before selecting a goal.
- **map_saver_cli failing on first save attempt**: `Failed to spin map subscription` — occurred because the save command was run before the robot had been driven at all, so `/map` had no published data yet. Fixed by confirming `/map` topic activity via `ros2 topic hz /map` before saving.

## Contents
- `maps/` — saved occupancy grid map (`.pgm` + `.yaml`) built via SLAM Toolbox
- Demo videos (hosted on Google Drive, linked below)

## Demo videos
- [TeleOp Recording — live SLAM map building](https://drive.google.com/file/d/1_spngtMJz-SQHmhkxse3aCIdbUCTf92c/view?usp=sharing)
- [Nav2 Path Recording — autonomous navigation on the built map](https://drive.google.com/file/d/1odA90gwVX3KcuC8RGn8VNMPGyRPbEK8S/view?usp=sharing)

## Contact
Sanju N Sebastian · linkedin.com/in/sanju-n-sebastian · sanju.n.sebastian@gmail.com
