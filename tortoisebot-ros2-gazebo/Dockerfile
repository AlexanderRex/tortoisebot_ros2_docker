FROM nvidia/cuda:11.7.1-base-ubuntu20.04

# Set environment variables to skip timezone selection
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# Minimal setup and locale configuration
RUN apt update \
    && apt install -y locales lsb-release curl gnupg2 tzdata \
    && ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    && locale-gen en_US en_US.UTF-8 \
    && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 \
    && echo "LANG=en_US.UTF-8" > /etc/default/locale

# Add the ROS 2 apt repository
RUN apt update \
    && apt install -y software-properties-common \
    && add-apt-repository universe \
    && apt update \
    && curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros2.list

# Install development tools and ROS tools
RUN apt update \
    && apt install -y \
    python3-pip \
    python3-pytest-cov \
    ros-dev-tools \
    && pip3 install -U \
    flake8-blind-except \
    flake8-builtins \
    flake8-class-newline \
    flake8-comprehensions \
    flake8-deprecated \
    flake8-docstrings \
    flake8-import-order \
    flake8-quotes \
    pytest-repeat \
    pytest-rerunfailures \
    setuptools

# Install ROS 2 Galactic and Gazebo ROS integration
RUN apt update \
    && apt install -y ros-galactic-desktop ros-galactic-gazebo-ros-pkgs \
    && echo "source /opt/ros/galactic/setup.bash" >> /root/.bashrc

# Create the workspace and clone the project repository
RUN mkdir -p /root/ros2_ws/src \
    && git clone -b ros2-galactic https://github.com/AlexanderRex/tortoisebot.git /root/ros2_ws/src/tortoisebot

# Remove all folders except tortoisebot_gazebo and tortoisebot_description
RUN find /root/ros2_ws/src/tortoisebot/* -maxdepth 0 -type d ! \( -name 'tortoisebot_gazebo' -o -name 'tortoisebot_description' \) -exec rm -rf {} +

# Install dependencies using rosdep
RUN apt update \
    && rosdep init \
    && rosdep update \
    && /bin/bash -c "source /opt/ros/galactic/setup.bash && cd /root/ros2_ws && rosdep install --from-paths src --ignore-src -y --skip-keys \"fastcdr rti-connext-dds-5.3.1 urdfdom_headers\""

# Build the workspace
RUN /bin/bash -c "source /opt/ros/galactic/setup.bash && cd /root/ros2_ws && colcon build --symlink-install"

# Source the setup script
RUN echo "source /root/ros2_ws/install/local_setup.bash" >> /root/.bashrc
