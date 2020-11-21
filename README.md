# Pan-Tilt Ros Melodic Bringup
A tutorial to bring-up Pan-Tilt Turret in ROS Melodic and Gazebo.

## Getting Started

### Prerequisites
1. Ros Melodic

    Install following instructions on http://wiki.ros.org/melodic/Installation.
2. ROS Packages.

    Install them via,
    ```
    sudo apt-get install ros-melodic-arbotix_python
    ```

### Installation
1. Clone the repositories,
    ```
    mkdir -p ~/pantilt_ws/src
    cd ~/pantilt_ws/src
    git clone https://github.com/RobotnikAutomation/widowx_turret.git
    ```
2. Build packages with `catkin_make` and source the workspace by,
    ```
    cd ..  # cd to workspace root dir.
    catkin_make
    source devel/setup.bash
    ```
3. Even though we can now import the robot's URDF to Gazebo and visualize it, we need some additional work to control the joints of the model: 

    1. First, create a `widowx_turret_gazebo` package,
        ```
        cd src/widowx_turret
        catkin_create_pkg widowx_turret_gazebo
        cd widowx_turret_gazebo 
        mkdir launch
        cd launch
        touch spawn_widowx_turret.launch widowx_turret_empty_world.launch  
        ```
    2. Copy the following contents to the created launch files,
    
        #### spawn_widowx_turret.launch:
        ```
        <?xml version="1.0"?>

        <launch>
            <!-- Load the Robot Description-->
            <include file="$(find widowx_turret_description)/launch/load_description.launch"/>

        <!-- Run a python script to the send a service call to gazebo_ros to spawn a URDF robot -->
        <node name="spawn_widowx_turret_model" pkg="gazebo_ros" type="spawn_model"
            args="-urdf -model widowx_turret -param robot_description"/>
            
            <!-- Launch widowx gazebo ros_control -->
            <include file="$(find widowx_turret_controller)/launch/widowx_turret_gazebo_controller.launch"/>

        </launch>
        ```
        #### widowx_turret_empty_world.launch:
        ```
        <?xml version="1.0"?>

        <launch>

        <arg name="world_name" default="worlds/empty.world"/>
        
        <include file="$(find gazebo_ros)/launch/empty_world.launch">
            <arg name="world_name" value="$(arg world_name)"/> <!-- world_name is wrt GAZEBO_RESOURCE_PATH environment variable -->
            <arg name="paused" value="false"/>
            <arg name="use_sim_time" value="true"/>
            <arg name="gui" value="true"/>
            <arg name="headless" value="false"/>
            <arg name="debug" value="false"/>
        </include>
        
        <include file="$(find widowx_turret_gazebo)/launch/spawn_widowx_turret.launch"/>

        </launch>
        ```
    3. Add controller configs for pan and tilt joint controllers: 
        
        First create a yaml file,
        ```
        roscd widowx_turret_controller/config
        touch widowx_control.yaml
        ```
        Then, copy the following contents:
        ```
        pan_controller:
            type: effort_controllers/JointPositionController
            joint: servo_pan_joint
            pid: {p: 20, i: 0.1, d: 1}

        tilt_controller:
            type: effort_controllers/JointPositionController
            joint: servo_tilt_joint
            pid: {p: 20, i: 0.1, d: 1}

        # Publish all joint states -----------------------------------
        joint_state_controller:
        type: joint_state_controller/JointStateController
        publish_rate: 50  
        ```
    4. Edit robot's URDF to use ros_control. 
        ```
        ```

5. 
3. Set `sim` to `True` within the launch file so that the turret uses its fake version.
3. Test the installation by,
    ```
    roslaunch segway_gazebo segway_empty_world.launch
    ```
