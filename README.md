# ros2-navigation-test
Testing ROS2 Navigation stack on with ros2 humble and gazebo harmonic.  \

## 1. Creating docker container from scratch     

- Pull ubuntu-22 ros2 humble image
    ```bash
    docker pull osrf/ros:humble-desktop
    ```

- Create a container using that image
    ```bash
    xhost +local:docker
    docker run -it \
    --name ros2-navigation-container \
    --net=host \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v ~/docker-workspace:/root/ \
    osrf/ros:humble-desktop
    ```

    ### Installing dependencies

    - Open container bash and install gazebo harmonic in it 
        ```bash
        docker start ros2-navigation-container && docker exec -it ros2-navigation-container bash
        apt-get update
        apt-get install curl lsb-release gnupg
        curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] https://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
        apt-get update
        apt-get install gz-harmonic
        gz sim --versions
        gz sim -v4
        ```

    - Install gz sim and ros interfacing packages
        ```bash
        sudo apt-get install ros-humble-ros-gzharmonic
        ```

    - Create a workspace and build the **bcr_bot** pkg  
        ```bash
        mkdir -p ~/bcr_ws/src
        cd ~/bcr_ws/src
        git clone -b ros2 https://github.com/ab31mohit/bcr_bot.git
        cd ~/bcr_ws/
        rosdep update
        rosdep install --from-paths src --ignore-src -r -y
        colcon build --packages-select bcr_bot
        echo "source ~/bcr_ws/install/setup.bash" >> ~/.bashrc
        source ~/.bashrc
        ```

    ### Running the simulation  

    - Launch the gazebo simulation 
        ```bash
        ros2 launch bcr_bot gz.launch.py \
            camera_enabled:=True \
            stereo_camera_enabled:=False \
            two_d_lidar_enabled:=True \
            position_x:=0.0 \
            position_y:=0.0  \
            orientation_yaw:=0.0 \
            odometry_source:=world \
            world_file:=small_warehouse.sdf
        ```  

    - Launch rviz2  
        ```bash
        ros2 launch bcr_bot rviz.launch.py
        ``` 

## 2. Using existing docker image for the project 

```bash
docker pull ab31mohit/ros2-humble:bcr-bot-simulation
docker images
xhost +local:docker
docker run -it --rm \
  --name bcr-bot-sim-container \
  --net=host \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/docker-workspace:/root/ \
  ab31mohit/ros2-humble:bcr-bot-simulation
```