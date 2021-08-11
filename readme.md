

# ros2 topic publishing by Teensy 4.1 using static library

## Masoud Iranmehr

This repository shows a simple Arduino example of uros publishing an integer into ROS2, using

* ROS2 foxy
* uros static library
* Teensy 4.1 as Hardware
* Platformio IDE



## How to run this project

### Install docker (for ubuntu)

	sudo apt-get update
	sudo apt-get install \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    gnupg \
	    lsb-release
    
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

	echo \
	  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
	  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	  
	 sudo apt-get update
	 sudo apt-get install docker-ce docker-ce-cli containerd.io      



#### Load docker image

    docker run -it --net=host -v /dev:/dev --privileged ros:foxy


### Install teensy_loader_cli

Just follow these steps for ubuntu users:

    cd /
    git clone https://github.com/PaulStoffregen/teensy_loader_cli 
    cd teensy_loader_cli
    sudo apt-get install libusb-dev
    make

#### Add Teensy rules to linux

For program Teensy you need these rules to be added to the linux:

    wget https://www.pjrc.com/teensy/00-teensy.rules
    sudo cp 00-teensy.rules /etc/udev/rules.d/

    sudo apt-get update
    sudo apt-get install -y gcc-arm-none-eabi

Please see this [link](https://www.pjrc.com/teensy/loader_linux.html).    

### Install cross-compiler for ARM

    sudo apt-get update
    sudo apt-get install -y gcc-arm-none-eabi

### How to compile uros static library


#### Install uros

    # Source the ROS 2 installation
    source /opt/ros/$ROS_DISTRO/setup.bash

    # Create a workspace and download the micro-ROS tools
    mkdir microros_ws
    cd microros_ws
    git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup

    # Update dependencies using rosdep
    sudo apt update && rosdep update
    rosdep install --from-path src --ignore-src -y

    # Install pip
    sudo apt-get install python3-pip

    # Build micro-ROS tools and source them
    colcon build
    source install/local_setup.bash

#### Creating the micro-ROS agent

    # Download micro-ROS-Agent packages
    ros2 run micro_ros_setup create_agent_ws.sh

    # Build step
    ros2 run micro_ros_setup build_agent.sh
    source install/local_setup.bash

#### Creating static library workspace

    ros2 run micro_ros_setup create_firmware_ws.sh generate_lib
    mkdir static_library_cfg
    cd static_library_cfg
    wget https://github.com/masoudir/teensy41_uros_example_publisher_with_static_library/blob/master/static_library_cfg/teensy4.cmake
    wget https://github.com/masoudir/teensy41_uros_example_publisher_with_static_library/blob/master/static_library_cfg/colcon.meta

    ros2 run micro_ros_setup build_firmware.sh /microros_ws/static_library_cfg/teensy4.cmake /microros_ws/static_library_cfg/colcon.meta

### How to create Platformio project

You need to install vscode in your host and then install remote & docker plugins to access docker folders. This automatically installs vscode-server in your docker. Then you need to install Platformio plugin inside vscode-server in your docker.

Then clone this repo:

    cd /
    git clone https://github.com/masoudir/teensy41_uros_example_publisher_with_static_library

    cd teensy41_uros_example_publisher_with_static_library

Then if we want to add our static library to the project, we need to remove the older one and then copy the newer one:

    rm -r /teensy41_uros_example_publisher_with_static_library/lib/microros/*
    rm /teensy41_uros_example_publisher_with_static_library/lib/libmicroros.a
    
    cd /teensy41_uros_example_publisher_with_static_library/lib/microros/
    wget https://github.com/masoudir/teensy41_uros_example_publisher_with_static_library/blob/master/lib/microros/default_transport.cpp
    wget https://github.com/masoudir/teensy41_uros_example_publisher_with_static_library/blob/master/lib/microros/micro_ros_arduino.h

    cp -r /microros_ws/firmware/build/include/* /teensy41_uros_example_publisher_with_static_library/lib/microros/
    cp /microros_ws/firmware/build/libmicroros.a /teensy41_uros_example_publisher_with_static_library/lib/

By default we have added this library, so you do not need to rebuilt that and copy them.    

Then build this project using platformio build in vscode. (You can use CLI commands instead, but we do not mention that in this page).


#### Flash the hex file to Teensy   

    /teensy_loader_cli/teensy_loader_cli --mcu=TEENSY41 -s /teensy41_uros_example_publisher_with_static_library/.pio/build/teensy41/firmware.hex

*Note: "-s" : Use sort reboot if device not online. Perform a sort reset request by searching for any Teensy running USB Serial code built by Teensyduino. A request to reboot is transmitted to the first device found.

#### Run uros agent

Open a new terminal:

    cd /microros_ws
    source /opt/ros/foxy/setup.bash
    source install/setup.bash

    ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0

If session is not established, while this agent is on, try to reset the microcontroller (reset in hardware). Becareful do not disconnect the serial port while agent is up.

#### How to read the topic that is published

Open a new terminal:

    source /opt/ros/foxy/setup.bash; source /opt/ros/foxy/local_setup.bash
    ros2 topic list
    ros2 topic echo /micro_ros_arduino_node_publisher


