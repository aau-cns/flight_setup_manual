# Setup the catkin workspace

```sh
# Generate a catkin workspace
$ mkdir -p catkin_ws/src
$ cd catkin_ws
$ catkin init
$ catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
$ cd src
# Clone and build packages
$ git clone -b ext_state_est https://github.com/aau-cns/mavlink-gbp-release.git mavlink
$ git clone -b ext_state_est https://github.com/aau-cns/mavros.git
$ cd ..
$ catkin build
```

# Build the PX4 Firmware

## Pull and pre-setup the PX4 Firmware Repository

```sh
$ git clone -b ext_state_est https://github.com/aau-cns/PX4-Autopilot.git px4
$ cd px4
```

## Build in Docker (Optional)

```sh
$ docker run -it --rm --network=host -v "$(pwd)":/source px4io/px4-dev-nuttx-bionic:latest
$ cd source
$ make px4_fmu-v5_default
```

# Build or Update Mavlink Messages

Mavlink is used in various places:

```sh
# For a catkin workspace
https://github.com/mavlink/mavlink-gbp-release.git
https://github.com/mavlink/mavros/
# For the building of mavlink messages
https://github.com/malink/mavlink.git
https://github.com/mavlink/c_library_v2.git
```

All repositories have CNS-adapted versions for the bridge and [MaRS](https://github.com/aau-cns/mars_lib) core message adaptations:

```sh
https://github.com/aau-cns/mavlink-gbp-release.git
https://github.com/aau-cns/mavros.git
https://github.com/aau-cns/mavlink.git
https://github.com/aau-cns/mavlink_c_library_v2.git
```

To *update*, mavlink-gbp-release, px4_mavros and mavlink its sufficient to find the version which is needed for the current PX4 version and rebase the internal branches. To add messages, you need to add message definitions to e.g. *message_definitions/common.xml* of mavlink-gbp-release and mavlink.
px4_mavros requires the addition of plugins, you can use the addition of the mars-core-state message as a template.

To update or add Mavlink definitions to the PX4 firmware you need to edit e.g. *mavlink_c_library_v2/message_definitions/common.xml* and add the message definition. Then compile *mavlink-gbp-release* for the pymavlink generator with catkin, you could do the same by directly compiling Mavlink. Enter the project folder which contains *mavgenerate.py* and run the following commands:

```sh
python3 -m pymavlink.tools.mavgen --lang=C --wire-protocol=2.0 --output=../../../mavlink_c_library_v2/ ../../../mavlink_c_library_v2/message_definitions/common.xml
python3 -m pymavlink.tools.mavgen --lang=C --wire-protocol=2.0 --output=../../../mavlink_c_library_v2/ ../../../mavlink_c_library_v2/message_definitions/standard.xml
```

Ensure that you compile the message entities that you edited. IMPORTANT note: If *common.xml* is changed, the *standard.xml* must be rebuilt because standard messages are based on common. Also, make that you build both results to the same output directory. The build script searches for existing messages and changes the generation depending on it.

After *mavlink_c_library_v2* is updated, update the submodule of this repository in the PX4 firmware:

```sh
cd px4_cns/mavlink/include/mavlink/v2.0
```

# Run Gazebo Headless Simulations in Docker

```sh
git clone -b ext_state_est https://github.com/aau-cns/PX4-Autopilot.git px4
docker run -it --privileged --network=host -v "$(pwd)":/source px4io/px4-dev-ros-melodic:latest bash
cd source
HEADLESS=1 make px4_sitl gazebo
```
