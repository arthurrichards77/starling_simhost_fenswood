#Starling Simhost Fenswood

This is _temporary_ to develop functionality to wrap into either the main Fenswood scenario image or into the new controller example template.

Created new intermediate image with the Fenswood stuff added to the image rather than mounted as a volume.  Goal is to avoid need for student users to download the Fenswood model source material.

Also used this as a platform to experiment with "manual" driving of the simulator using ROS2 command line utilities.

## Running the simulation

Use the provided compose file: `docker-compose pull` then `docker-compose up`.

Should be able to see the Gazebo environment on [localhost:8080](http://localhost:8080) and [connect Foxglove to ws:localhost:9090](https://studio.foxglove.dev/?ds=rosbridge-websocket&ds.url=ws%3A%2F%2Flocalhost%3A9090).

Can also connect QGroundControl to `tcp:localhost:5761`.

## Flying the simulator through ROS command line tools

The following command will get you a command prompt in the MAVROS container with ROS2 all set up:
```
docker exec -it starling_simhost_fenswood_mavros_1 /bin/bash -c "source /opt/ros/foxy/setup.bash;/bin/bash"
```
Use the utility batch file ssh_mavros.bat on Windows.

Change mode:
```
ros2 service call /vehicle_1/mavros/set_mode mavros_msgs/srv/SetMode "{custom_mode: "GUIDED"}"
```
Arm:
```
ros2 service call /vehicle_1/mavros/cmd/arming mavros_msgs/srv/CommandBool "{value: True}"
```
Take-off:
```
ros2 service call /vehicle_1/mavros/cmd/takeoff mavros_msgs/srv/CommandTOL "{altitude: 20.0}"
```
Monitor:
```
ros2 topic echo /vehicle_1/mavros/global_position/global
```
Note: monitoring may not work if QGroundControl hasn't been connected as MAVLINK data streams have to be requested.

To move, run the following (note: for reasons I don't understand, altitude relative seems to be what's published, -100):
```
ros2 topic pub --once /vehicle_1/mavros/setpoint_position/global geographic_msgs/msg/GeoPoseStamped "{pose:{position:{latitude: 51.423, longitude: -2.671, altitude: 120.0}}}"
```
To move the camera use the command below, with data in range 0.0 (forwards) to 1.57 (straight down):
```
/ros_ws# ros2 topic pub --times 5 /vehicle_1/gimbal_tilt_cmd std_msgs/msg/Float32 "{data: 1.0}"
```
Note: it seems to need sending several times to ensure relialbe response.

To land in current location, run:
```
ros2 service call /vehicle_1/mavros/cmd/land mavros_msgs/srv/CommandTOL
```
Or to go home and land, change mode to "RTL" (Return To Land):
```
ros2 service call /vehicle_1/mavros/set_mode mavros_msgs/srv/SetMode "{custom_mode: "RTL"}"
```
Exit the container prompt using Ctrl+D.

## Building the image

On Linux use `make build` then `make push`.

On Windows without Make, use the utilities `build.bat` and `push.bat`.

Note: push currently goes to arthurrichards77 repository on Docker hub.  Long term, this should be migrated to Starling.
