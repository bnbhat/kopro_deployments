FROM nvidia/cuda:12.1.0-devel-ubuntu22.04

ARG ZED_SDK_MAJOR=4
ARG ZED_SDK_MINOR=0

ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}compute,video,utility

# Setup timezone and CUDA version
RUN echo "Europe/Paris" > /etc/localtime ; echo "CUDA Version 12.1.0" > /usr/local/cuda/version.txt

# Install essential packages
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    lsb-release \
    wget \
    less \
    udev \
    sudo \
    zstd \
    build-essential \
    cmake \
    git \
    python3 \
    python3-pip \
    libpng-dev \
    libgomp1 \
    bash-completion \
    dirmngr \
    gnupg2 \
    lsb-release \
    software-properties-common

# Install Python packages
RUN python3 -m pip install numpy opencv-python

# Install ZED SDK
RUN wget -q -O ZED_SDK_Linux_Ubuntu22.run https://download.stereolabs.com/zedsdk/${ZED_SDK_MAJOR}.${ZED_SDK_MINOR}/cu121/ubuntu22 && \
    chmod +x ZED_SDK_Linux_Ubuntu22.run ; ./ZED_SDK_Linux_Ubuntu22.run -- silent skip_cuda && \
    ln -sf /lib/x86_64-linux-gnu/libusb-1.0.so.0 /usr/lib/x86_64-linux-gnu/libusb-1.0.so && \
    rm ZED_SDK_Linux_Ubuntu22.run && \
    rm -rf /var/lib/apt/lists/*

# Setup ROS2 repository and install ROS2 Humble
RUN apt-get update && apt-get install -y curl gnupg2 lsb-release && \
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2.list && \
    apt-get update && apt-get install -y ros-humble-desktop ros-dev-tools ros-humble-cv-bridge ros-humble-vision-opencv

RUN python3 /usr/local/zed/get_python_api.py

# Download and optimize the model
RUN ZED_Diagnostic -ais 3 

# Create ROS2 workspace
WORKDIR /ros2_ws
RUN mkdir src
WORKDIR /ros2_ws/src

# Clone repositories
RUN git clone https://github.com/bnbhat/arc_interfaces.git
RUN git clone https://github.com/bnbhat/zed_ros2_driver.git

# Build the workspace
WORKDIR /ros2_ws
COPY ./test.svo .
RUN /bin/bash -c 'source /opt/ros/humble/setup.bash && colcon build --symlink-install'

# Set up .bashrc for the root user
RUN echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc && \
    echo 'source /ros2_ws/install/setup.bash' >> ~/.bashrc && \
    echo 'export ROS_DOMAIN_ID=0' >> ~/.bashrc

# Clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

CMD ["/bin/bash", "-lc", "python3 ./src/zed_ros2_driver/zed_driver/zed_ros_driver.py"]