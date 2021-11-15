# PREMAPPING ENVIRONMENT

## RTAB-MAP SETUP
For official documentation please refer to this [ros-wiki](http://wiki.ros.org/rtabmap_ros/Tutorials/HandHeldMapping)


## Setup Environment
Download Dependencies (`rtabmap-ros` and `imu-filter`)
`sudo apt install -y ros-melodic-rtabmap-ros ros-melodic-imu-filter-madgwick`


## Realsense D435i setup
If you are using a `realsense D435i` to test the module hence you will need `realsense2-camera`.

```bash
sudo apt-get install ros-melodic-realsense2-camera ros-melodic-realsense2-description
cd ~/
mkdir -p realsense_ws/src && cd realsense_ws/src
git clone git@github.com:IntelRealSense/realsense-ros.git
cd ..
catkin build
roslaunch realsense2_camera rs_camera.launch #To test whether it is working
```

**However** to launch the module and get it ready for `RTAB-MAP`, edit `rs_aligned_depth.launch` and add **2** lines
```
<launch>
...
<!-- this line -->
<arg name="unite_imu_method"    default="linear_interpolation"/>
...
<include file="$(find realsense2_camera)/launch/includes/nodelet.launch.xml">
...
<!-- and this line -->
	<arg name="unite_imu_method"         value="$(arg unite_imu_method)"/>
```

- I have tested with the `realsense D435i` on the `Xavier NX`, and there are some cpu limitation sending over ROS network setup. Hence, run it and ROS locally. You can use the tested `launch` file attached in `/files`.

After which launch `roslaunch realsense2_camera rs_aligned_depth.launch`.

- Since it is offline, when you are playing the bag and you want to truncate the bag, use `rosbag filter input.bag output.bag "t.secs <= 1284703931.86"` the time is corresponding to the clock time.


## IMU FILTER MADGWICK
Launch the `madgwick filter` for IMU
```bash
rosrun imu_filter_madgwick imu_filter_node \
	_use_mag:=false \
        _publish_tf:=false \
        _world_frame:="enu" \
        /imu/data_raw:=/camera/imu \
        /imu/data:=/rtabmap/imu
```


## RTAB-MAP
Launch the `RTABMAP`
```bash
roslaunch rtabmap_ros rtabmap.launch \
    rtabmap_args:="--delete_db_on_start --Optimizer/GravitySigma 0.3" \
    depth_topic:=/camera/aligned_depth_to_color/image_raw \
    rgb_topic:=/camera/color/image_raw \
    camera_info_topic:=/camera/color/camera_info \
    approx_sync:=false \
    wait_imu_to_init:=true \
    imu_topic:=/rtabmap/imu
```


## Post Processing using RTABMAP GUI
1. Pause the scan
2. Click on the Tools option and there should be `Post-Processing` option.
3. There should be `Sparse Bundle Adjustment (SBA)`, this would optimize the camera pose throughout the path and **readjust** the point cloud
4. Under `File` choose `Export 3D Clouds`, and load the `rtabmap_gui2.ini` from `/files`, this has `cloud filtering` when exporting to `.ply`

- After which you can use https://github.com/aaravrav142/ply_publisher to publish the pointclouds, remember to create a workspace for the package
- Sample to publish the point clouds would be as shown below
```
rosrun ply_publisher ply_publisher \
    _file_path:=/home/**user**/mesh/corridor1.ply \
    _topic:=/point_cloud \
    _frame:=/map \
    _rate:=1
```
- The next step would be to create a `transformation (tf)` to convert the point clouds to align with the `/map` or `/world`



## Post Processing using PyTorch
After receiving the pointcloud data under `rtabmap/cloud_map`, launch `rosrun pcl_ros pointcloud_to_pcd input:=/rtabmap/cloud_map`.

Still having `PyTorch` related issues when using this package hence I have not tried it out yet
```
git clone git@github.com:ranahanocka/point2mesh.git
```
