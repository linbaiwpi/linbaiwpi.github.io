# Velodyne driver running on PYNQ on Xilinx MPSoc ZCU104 platform
Date: 2019-10-31

[**Link to project repository**](https://github.com/linbaiwpi/VLP16_driver_on_PYNQ)

## Purpose
The purpose of this repository is to connect the Velodyne VLP-16 LiDAR to our ZCU104 board so that PYNQ can process the point cloud directly. Considering the process speed of Python, C++ is a good candidate for driver. Therefore, we have modified the Velodyne driver supplied by the [this link](https://github.com/ros-drivers/velodyne) to get rid of the ROS operating system. The data port number(DATA_PORT_NUMBER in src/udp_input.h) must consist to LiDAR setting.

The principle is, 1. compile the C++ code as a dynamic library of Linux, 2. Use Python library ctypes to capture point cloud into Python environment, 3. plot the point cloud.

This code is specific to VLP-16, but it's very easy to adapt the code to HDL-32 or HDL-64E.
### Requirement
- g++
- numpy
- matplotlib


## Status
Currently, the code has been tested on the Linux desktop version.

## How to use

```sh
$ cd ./src
$ g++ -std=c++11 run.cpp udp_input.cpp VelodyneDriver.cpp calibration.cpp convert.cpp  pointcloudXYZIR.cpp rawdata.cpp -fPIC -shared -o ../libvelodynedriver.so
$ cd ..
$ python pc_captue.py
```

<figure class="whole">
	<img src="/image/zcu104_tut/tut3_1.png">
	<figcaption>Demo point cloud plot</figcaption>
</figure>

## Todo list
**[x]** Driver modification

**[x]** Verification on desktop linux

**[ ]** Test on PYNQ
