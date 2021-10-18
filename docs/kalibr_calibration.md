# KALIBR CALIBRATION
For official documentation please refer to this [wiki](https://github.com/ethz-asl/kalibr/wiki)

- *Installation* (abit outdated) [here](https://github.com/ethz-asl/kalibr/wiki/installation) **Do not use CDE package** don't be lazy and build it
- *Camera-calibration* [here](https://github.com/ethz-asl/kalibr/wiki/multiple-camera-calibration)
- *Supported camera models* [here](https://github.com/ethz-asl/kalibr/wiki/supported-models)
- *Calibration targets* [here](https://github.com/ethz-asl/kalibr/wiki/calibration-targets)

## Setup Environment
```bash
# Install dependencies
sudo apt-get install -y python-setuptools python-rosinstall ipython libeigen3-dev libboost-all-dev doxygen libopencv-dev ros-$ROS_DISTRO-vision-opencv ros-$ROS_DISTRO-image-transport-plugins ros-melodic-cmake-modules software-properties-common libpoco-dev python-matplotlib python-scipy python-git python-pip ipython libtbb-dev libblas-dev liblapack-dev python-catkin-tools libv4l-dev python-igraph

mkdir -p ~/kalibr_ws/src # Kalibr Workspace
mkdir ~/calibration # Data and bag files

# Setup Catkin configuration and install additional dependencies
cd kalibr_ws
catkin config --extend /opt/ros/melodic --merge-devel -DCMAKE_BUILD_TYPE=Release
pip install Pillow
sudo apt-get -y install libv4l-dev

# Git clone
cd src
git clone https://github.com/ethz-asl/Kalibr.git

# Build the package
cd ..
catkin build -DCMAKE_BUILD_TYPE=Release -j3
```

**Remember to always source** using `source devel/setup.bash`, this will give you mostly 3 options for the next step. `kalibr_create_target_pdf`, `kalibr_calibrate_cameras` and `kalibr_calibrate_imu_camera`

```bash
# Format of making a kalibr target
# kalibr_create_target_pdf --type apriltag --nx [NUM_COLS] --ny [NUM_ROWS] --tsize [TAG_WIDTH_M] --tspace [TAG_SPACING_PERCENT]
# But you can just find preset ones in *files*
```

## Calibrating Cameras
This step assumes that you have a `rosbag` file. Use `rosbag info FILENAME.bag` to find the topics, you should have **3 topics** in total (make sure that `cam1` and `cam2` have the same number of messages).

Sample for **D435i** bag file output (`/camera/imu` is from `unite_imu_method='linear-interpolation'` in `rs_camera.launch`)
```
types:       sensor_msgs/Image [060021388200f6f0f447d0fcd9c64743]
             sensor_msgs/Imu   [6a62c6daae103f4ff57a132d6f95cec2]
topics:      /camera/imu                     10529 msgs    : sensor_msgs/Imu  
             /camera/infra1/image_rect_raw    1579 msgs    : sensor_msgs/Image
             /camera/infra2/image_rect_raw    1579 msgs    : sensor_msgs/Image
```

Assuming you have places your bag and your april tag inside `calibration` folder, format for the calibration is
```
# Fromat
kalibr_calibrate_cameras \
--topics [cam1/image] [cam2/image] \
--models [model-cam1] [model-cam2] \
--bag [path to bag] \
--target [path to april tag yaml]
``` 

Sample for **D435i** 
```
kalibr_calibrate_cameras --topics /camera/infra1/image_rect_raw /camera/infra2/image_rect_raw --models pinhole-radtan pinhole-radtan --bag ~/calibration/camera-imu-data1.bag --target ~/calibration/april_5x5.yaml
```
