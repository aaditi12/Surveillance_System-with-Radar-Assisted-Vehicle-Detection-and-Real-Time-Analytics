# Surveillance_System-with-Radar-Assisted-Vehicle-Detection-and-Real-Time-Analytics
Here's the full run sequence, assuming your workspace is already set up the way your other assignments have been (`~/assignment_ws` or similar on `pi-node1`):

## 1. Build & source

```bash
cd ~/assignment_ws
colcon build --symlink-install
source install/setup.bash
```

## 2. One-command launch (does everything)

```bash
ros2 launch testbed_bringup multi_robot_surveillance.launch.py
```

This spawns all 4 ground robots + drone, starts the 4 patrol loops, starts the drone's round-robin surveillance, starts the dashboard, and opens RViz.

**Tunable args:**
```bash
ros2 launch testbed_bringup multi_robot_surveillance.launch.py \
  robot_speed:=0.6 \
  drone_altitude:=12.0 \
  dwell:=5.0 \
  web_port:=9090 \
  rviz:=false
```

## 3. Open the radar dashboard

```
http://localhost:8080/
```
(or `http://<pi-node1-ip>:8080/` if you're viewing it from another machine on the network)

## 4. Watch the data in a terminal instead of the web UI

```bash
ros2 topic echo /surveillance/station_report
ros2 topic echo /surveillance/current_target
ros2 topic echo /surveillance/robots_report
```

## 5. If you want to run pieces separately (for debugging)

```bash
# just the world + 4 robots + drone, no patrol/survey yet
ros2 launch testbed_gazebo spawn_playground.launch.py

# spawn one robot manually
ros2 launch testbed_gazebo spawn_ground_robot.launch.py robot_name:=ground_robot1 x:=14.0 y:=0.0

# spawn the drone manually
ros2 launch testbed_gazebo spawn_drone.launch.py robot_name:=drone x:=0.0 y:=0.0 z:=3.0

# patrol one robot by hand
ros2 run testbed_navigation ground_patrol.py -- 12 -1 16 -1 16 1 12 1 --namespace ground_robot1 --speed 0.4 --loop

# run the drone survey node standalone (robots must already be moving)
ros2 run testbed_navigation drone_multi_survey.py \
  --ground-namespaces ground_robot1,ground_robot2,ground_robot3,ground_robot4 \
  --callsigns GRND01,GRND02,GRND03,GRND04 \
  --altitude 10.0 --dwell 8.0

# run just the dashboard against robots that are already up
ros2 run testbed_navigation multi_surveillance_station.py \
  --ground-namespaces ground_robot1,ground_robot2,ground_robot3,ground_robot4 \
  --web-port 8080

# open RViz on its own
rviz2 -d install/testbed_description/share/testbed_description/rviz/multi_robot_surveillance.rviz
```

## 6. Shut down

`Ctrl+C` in the launch terminal — the patrol/survey nodes stop their robots cleanly (they publish a zero `Twist` on shutdown) before Gazebo exits.

If you're on the Pi over SSH with the `xterm` multi-terminal setup you were using before, each of the step-5 commands is a good candidate for its own `xterm` pane if you want to watch individual node logs instead of the interleaved single-launch output.
