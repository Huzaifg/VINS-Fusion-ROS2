#=============================================================================
## PROJECT CHRONO - http://projectchrono.org
##
## Copyright (c) 2023 projectchrono.org
## All rights reserved.
##
## Use of this source code is governed by a BSD-style license that can be found
## in the LICENSE file at the top level of the distribution and at
## http://projectchrono.org/license-chrono.txt.
##
## =============================================================================
## Authors: Thomas Liang, Huzaifa Unjhawala, Harry Zhang
## =============================================================================
FROM uwsbel/ros:latest

#####################################################
# Evironmental variables
#####################################################
ENV DISPLAY=:1 \
    VNC_PORT=5901 \
    NO_VNC_PORT=6901 \
    HOME=/sbel \
    TERM=xterm \
    STARTUPDIR=/dockerstartup \
    NO_VNC_HOME=/sbel/noVNC \
    DEBIAN_FRONTEND=noninteractive \
    VNC_COL_DEPTH=24 \
    VNC_RESOLUTION=1600x900 \
    VNC_PW=sbel
EXPOSE $VNC_PORT $NO_VNC_PORT

RUN apt update && apt install -y locales && locale-gen "en_US.UTF-8"
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'


SHELL ["/bin/bash", "-c"]

#####################################################
# Delete existing Chrono in iros2024
#####################################################
RUN rm -rf /sbel/Desktop/chrono

#####################################################
# Build Chrono and Install
#####################################################
ADD buildChrono.sh /
# ADD chrono-internal $HOME/Desktop/chrono
RUN chmod +x /buildChrono.sh && bash /buildChrono.sh


#####################################################
# Startup and Cleanup
#####################################################
ADD ./src/ $HOME/src/
ADD ./desktop/ $HOME/Desktop/
ADD bashrc $HOME/.bashrc
# RUN mkdir $HOME/Desktop/chrono/chrono_sensor_ros_node
# ADD ./chrono_sensor_ros_node/ $HOME/Desktop/chrono/chrono_sensor_ros_node
RUN chmod a+x $HOME/src/vnc_startup.sh $HOME/src/wm_startup.sh \
    && rm -rf /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 /usr/lib/x86_64-linux-gnu/libcuda.so.1 /usr/lib/x86_64-linux-gnu/libcudadebugger.so.1
    # && mkdir $HOME/ros-src/image_subscriber/
# ADD streamer.py $HOME/ros-src/image_subscriber/
WORKDIR /sbel

#####################################################
# Install Ceres Solver 
#####################################################
RUN apt-get update && apt-get install -y cmake libgoogle-glog-dev \
    && wget http://ceres-solver.org/ceres-solver-2.1.0.tar.gz \
    && tar zxf ceres-solver-2.1.0.tar.gz -C /tmp/ \
    && cd /tmp/ceres-solver-2.1.0 \
    && mkdir build && cd build \
    && cmake -DEXPORT_BUILD_DIR=ON \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DUSE_CUDA=ON \
    -DBUILD_TESTING=OFF \
    -DBUILD_EXAMPLES=OFF \
    -DTBB=OFF \
    ../ \
    && make -j $(nproc) \
    && make install -j $(nproc) \
    && rm -rf /tmp/ceres-solver-2.1.0 /tmp/ceres-solver-2.1.0.tar.gz

#####################################################
# Install OpenCV
#####################################################
RUN pip install opencv-python


WORKDIR /sbel

#####################################################
# Clone VINS-Fusion-ROS2
#####################################################

RUN mkdir ros2_ws && cd ros2_ws \
    && mkdir src && cd src \
    && git clone -b no_cuda https://github.com/Huzaifg/VINS-Fusion-ROS2.git

WORKDIR /sbel
ENTRYPOINT ["/sbel/src/vnc_startup.sh"]
