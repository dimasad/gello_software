FROM ros:humble-ros-base 

RUN apt-get update && apt-get install -y \
  # Install libfranka build depencencies
  build-essential \
  cmake \
  git \
  libeigen3-dev \
  libfmt-dev \
  libpoco-dev \
  ros-humble-pinocchio \
  ros-humble-rmw-cyclonedds-cpp \
  # Install missing franka_ros2 build dependencies (workaround franka_ros2 issue 160)
  ros-humble-controller-interface \
  ros-humble-hardware-interface \
  # Install ros packages
  python3-colcon-common-extensions \
  python3-colcon-mixin \
  python3-pip \
  python3-vcstool \
  && rm -rf /var/lib/apt/lists/*

# Install robotiq gripper
ARG ROS2_ROBOTIQ_GRIPPER_COMMIT_HASH="2ff85455d4b9f973c4b0bab1ce95fb09367f0d26"
RUN /bin/bash -c "source /opt/ros/humble/setup.bash && \
  mkdir tmp_build && \
  cd tmp_build && \
  git clone https://github.com/PickNikRobotics/ros2_robotiq_gripper.git && \
  cd ros2_robotiq_gripper && git checkout ${ROS2_ROBOTIQ_GRIPPER_COMMIT_HASH} && cd .. && \
  sed -i 's/kGripperMaxSpeed = 0.150;/kGripperMaxSpeed = 1.0;/g' ros2_robotiq_gripper/robotiq_driver/src/hardware_interface.cpp && \
  vcs import . --input ros2_robotiq_gripper/ros2_robotiq_gripper.humble.repos && \
  apt-get update && \
  rosdep update && \
  rosdep install --from-paths . --ignore-src -r -y && \
  colcon build --install-base /opt/ros/humble/franka --cmake-args -DCMAKE_BUILD_TYPE=Release && \
  cd .. && \
  rm -rf tmp_build && \
  rm -rf /var/lib/apt/lists/*"

# Install rqt reconfigure from source (for array parameters)
RUN /bin/bash -c "source /opt/ros/humble/setup.bash && \
  mkdir -p /tmp/rqt_reconfigure && cd /tmp && \
  git clone https://github.com/ros-visualization/rqt_reconfigure.git --branch humble && \
  cd rqt_reconfigure && \
  apt-get update && \
  rosdep update && \
  rosdep install --from-paths . --ignore-src -r -y && \
  colcon build --install-base /opt/ros/humble/franka --cmake-args -DCMAKE_BUILD_TYPE=Release && \
  cd .. && \
  rm -rf /tmp/rqt_reconfigure && \
  rm -rf /var/lib/apt/lists/*"

# Install libfranka, franka_ros2 and franka_description
# Note: The versions need to be set according to your robot's system version
# https://franka.de/fr3-compatibility-matrix
# Adjust these parameters in the docker-compose.yml or in a .env file
ARG LIBFRANKA_VERSION
ARG FRANKA_ROS2_VERSION
ARG FRANKA_DESCRIPTION_VERSION

RUN /bin/bash -c 'source /opt/ros/humble/setup.bash && \
mkdir -p /tmp/franka_ros2 && cd /tmp/franka_ros2 && \
  git clone --recursive https://github.com/frankarobotics/libfranka --branch ${LIBFRANKA_VERSION} && \
  git clone --recursive https://github.com/frankarobotics/franka_ros2.git --branch ${FRANKA_ROS2_VERSION} && \
  git clone --recursive https://github.com/frankarobotics/franka_description.git --branch ${FRANKA_DESCRIPTION_VERSION} && \
  apt-get update && \
  rosdep update && \
  rosdep install --from-paths . --ignore-src -r -y && \
  rm -rf /var/lib/apt/lists/* && \
  colcon build --install-base /opt/ros/humble/franka --cmake-args -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=off  && \
  cd .. && \ 
  rm -rf /tmp/franka_ros2 && \
  echo "source /opt/ros/humble/franka/setup.bash" >> ~/.bashrc && \
  echo "source /opt/ros/humble/franka/setup.sh" >> ~/.profile'
