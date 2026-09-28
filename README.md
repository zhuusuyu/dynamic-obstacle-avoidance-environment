# dynamic-obstacle-avoidance-environment
# TurtleBot3 Dynamic Pedestrian Environments

A ROS 1 and Gazebo Classic simulation package providing three reproducible dynamic-pedestrian environments for TurtleBot3 social navigation, collision avoidance, and reinforcement-learning evaluation.

The package includes animated pedestrian actors, LiDAR-visible collision proxies, enclosed test arenas, and ready-to-use ROS launch files.

## Features

- Three deterministic dynamic-pedestrian scenarios
- Support for TurtleBot3 Burger, Waffle, and Waffle Pi
- Animated Gazebo pedestrian actors
- Invisible kinematic proxies that make pedestrians detectable by the robot's LiDAR
- Standard TurtleBot3 ROS interfaces such as `/cmd_vel`, `/scan`, and `/odom`
- GUI and headless execution
- No external Gazebo models required beyond standard ROS/Gazebo resources
- Apache-2.0 license

## Environments

All three environments use an enclosed `8 m × 8 m` arena with walls at approximately `x/y = ±4 m`.

| Environment | Launch file | Pedestrians | Robot initial pose | Description |
|---|---|---:|---|---|
| Single Opposite Pedestrian | `social_teacher_wall_1ped_opposite.launch` | 1 | `(-3.5, 0.0, 0.0)` | One pedestrian repeatedly walks between opposite sides of the arena, producing a direct encounter with the robot. |
| Eight-Pedestrian Mixed Traffic | `social_teacher_wall_8ped_opposite.launch` | 8 | `(-3.5, 0.0, 0.0)` | Eight pedestrians follow deterministic boundary, rectangular, and diagonal trajectories with different loop periods. |
| Eight-Pedestrian Orbit Validation | `social_orbit_8ped_validation.launch` | 8 | `(-3.2, 0.0, 0.0)` | Four pedestrians orbit outer obstacles while four pedestrians move on an inner circular path around the arena center. |

### Single Opposite Pedestrian

The pedestrian starts at `(3.0, 0.0)` and walks toward `(-3.0, 0.0)`. It pauses while turning around and then returns to its initial position.

- Path length per direction: `6 m`
- Complete loop period: `142 s`
- Intended use: simple head-on interaction and collision-avoidance validation

### Eight-Pedestrian Mixed Traffic

Eight pedestrians move along different deterministic routes:

- Four pedestrians traverse the outer sides of the arena
- Four pedestrians follow internal rectangular or diagonal paths
- Loop periods range from approximately `120 s` to `184 s`
- Intended use: dense multi-pedestrian navigation and generalization evaluation

### Eight-Pedestrian Orbit Validation

All eight pedestrians move clockwise at approximately `0.08 m/s`.

- Four outer trajectories have radius `1.0 m`
- Four inner trajectories have radius `1.5 m`
- Outer trajectory period: approximately `78.54 s`
- Inner trajectory period: approximately `117.81 s`
- Static obstacles include one central cube and four outer cylinders
- Intended use: continuous curved-motion and multi-obstacle avoidance evaluation

## Pedestrian Representation

Gazebo actors provide the visible pedestrian animation. However, animated actors are not always detected reliably by simulated ray sensors.

To make pedestrians visible to the TurtleBot3 LiDAR, each actor is paired with an invisible kinematic proxy:

```text
Animated Gazebo actor
          │
          │ pose from /gazebo/model_states
          ▼
scan_visible_pedestrians.py
          │
          │ /gazebo/set_model_state
          ▼
Invisible LiDAR collision proxy
```

The proxy has a `0.5 m × 0.5 m` footprint and continuously follows the corresponding actor. The robot therefore observes moving pedestrians through the standard `/scan` topic.

## Requirements

The package was developed and tested with:

- Ubuntu 20.04
- ROS Noetic
- Gazebo Classic 11
- Python 3.8
- TurtleBot3 ROS packages
- Catkin workspace

Install the required ROS packages:

```bash
sudo apt update
sudo apt install \
  ros-noetic-gazebo-ros-pkgs \
  ros-noetic-turtlebot3-description \
  ros-noetic-xacro
```

A complete ROS Noetic desktop installation is recommended.

## Installation

Clone the repository into the `src` directory of a Catkin workspace:

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone https://github.com/YOUR_USERNAME/turtlebot3_dynamic_pedestrians.git
```

Build and source the workspace:

```bash
cd ~/catkin_ws
catkin_make
source devel/setup.bash
```

Select a TurtleBot3 model:

```bash
export TURTLEBOT3_MODEL=burger
```

To set it permanently:

```bash
echo 'export TURTLEBOT3_MODEL=burger' >> ~/.bashrc
source ~/.bashrc
```

> Only one ROS package named `turtlebot3_dynamic_pedestrians` should exist in the workspace. Duplicate copies will cause Catkin package-name conflicts.

## Usage

Each command starts Gazebo, spawns the selected TurtleBot3 model, creates the pedestrians, and starts the LiDAR-proxy controllers.

### 1. Single opposite pedestrian

```bash
roslaunch turtlebot3_dynamic_pedestrians \
  social_teacher_wall_1ped_opposite.launch
```

### 2. Eight-pedestrian mixed traffic

```bash
roslaunch turtlebot3_dynamic_pedestrians \
  social_teacher_wall_8ped_opposite.launch
```

### 3. Eight-pedestrian orbit validation

```bash
roslaunch turtlebot3_dynamic_pedestrians \
  social_orbit_8ped_validation.launch
```

Stop the current Gazebo process before launching another environment.

### Run without the Gazebo GUI

```bash
roslaunch turtlebot3_dynamic_pedestrians \
  social_teacher_wall_8ped_opposite.launch gui:=false
```

### Override the robot initial pose

```bash
roslaunch turtlebot3_dynamic_pedestrians \
  social_teacher_wall_1ped_opposite.launch \
  x_pos:=-3.5 y_pos:=0.0 yaw:=0.0
```

### Display the laser visualization

The orbit environment provides a `laser_visual` argument:

```bash
roslaunch turtlebot3_dynamic_pedestrians \
  social_orbit_8ped_validation.launch \
  laser_visual:=true
```

## Launch Arguments

| Argument | Default | Description |
|---|---|---|
| `model` | `$TURTLEBOT3_MODEL` | TurtleBot3 model: `burger`, `waffle`, or `waffle_pi` |
| `x_pos` | Environment-dependent | Initial robot x-coordinate |
| `y_pos` | `0.0` | Initial robot y-coordinate |
| `z_pos` | `0.0` | Initial robot z-coordinate |
| `yaw` | `0.0` | Initial robot heading in radians |
| `gui` | `true` | Enable or disable the Gazebo client |
| `laser_visual` | `false` | Display laser rays; available in the orbit environment |

## ROS Interfaces

The spawned TurtleBot3 uses the standard ROS interfaces:

| Interface | Type | Description |
|---|---|---|
| `/cmd_vel` | `geometry_msgs/Twist` | Robot velocity command |
| `/scan` | `sensor_msgs/LaserScan` | LiDAR observations |
| `/odom` | `nav_msgs/Odometry` | Robot odometry |
| `/gazebo/model_states` | `gazebo_msgs/ModelStates` | Actor and model poses |
| `/gazebo/set_model_state` | `gazebo_msgs/SetModelState` | Updates pedestrian proxy poses |

This package provides the simulation environments only. Navigation policies, planners, reward functions, goal managers, and evaluation scripts should be implemented separately.

## Package Structure

```text
turtlebot3_dynamic_pedestrians/
├── launch/
│   ├── social_orbit_8ped_validation.launch
│   ├── social_teacher_wall_1ped_opposite.launch
│   └── social_teacher_wall_8ped_opposite.launch
├── models/
│   ├── edge_goal_marker/
│   ├── pedestrian_actor_assets/
│   ├── pedestrian_lidar_proxy/
│   └── pedestrian_scan_box/
├── scripts/
│   └── scan_visible_pedestrians.py
├── worlds/
│   ├── social_orbit_8ped_validation.world
│   ├── social_wall_1ped_opposite.world
│   └── social_wall_8ped_opposite.world
├── CMakeLists.txt
├── package.xml
└── README.md
```

## Reproducibility and Limitations

- Pedestrian trajectories are deterministic and defined directly in the SDF world files.
- Pedestrians do not react to the robot or to one another.
- The environments use Gazebo Classic actors rather than a social-force or crowd simulator.
- The LiDAR proxies follow the visual actors through `/gazebo/set_model_state`.
- Different TurtleBot3 models have different footprints and sensor configurations, which may affect evaluation results.
- The package targets ROS 1 and Gazebo Classic; it has not been ported to ROS 2 or modern Gazebo.

## Acknowledgments

The pedestrian animation files `walk.dae` and `moonwalk.dae` are unmodified copies of the media assets distributed with Gazebo 11.

This package also depends on the TurtleBot3 robot descriptions provided by ROBOTIS.

## License

The package is released under the Apache License 2.0.

The bundled Gazebo animation assets retain their original Gazebo licensing and attribution.
