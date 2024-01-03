# tros create deb package Dockerfile
FROM ubuntu:focal

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends tzdata && \
    rm -rf /var/lib/apt/lists/*

# install packages
RUN apt-get update && apt-get install -q -y --no-install-recommends \
    dirmngr \
    gnupg2 \
    && rm -rf /var/lib/apt/lists/*

# setup sources.list
RUN echo "deb http://snapshots.ros.org/foxy/final/ubuntu focal main" > /etc/apt/sources.list.d/ros2-snapshots.list

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 4B63CF8FDE49746E98FA01DDAD19BAB3CBF125EA

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV ROS_DISTRO foxy

# install ros2 packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-foxy-ros-core=0.9.2-1* \
    && rm -rf /var/lib/apt/lists/*

# install bootstrap tools
RUN apt-get update && apt-get install --no-install-recommends -y \
    build-essential \
    git \
    python3-colcon-common-extensions \
    python3-colcon-mixin \
    python3-rosdep \
    python3-vcstool \
    && rm -rf /var/lib/apt/lists/*

# bootstrap rosdep
RUN rosdep init && \
  rosdep update --rosdistro $ROS_DISTRO

# setup colcon mixin and metadata
RUN colcon mixin add default \
      https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml && \
    colcon mixin update && \
    colcon metadata add default \
      https://raw.githubusercontent.com/colcon/colcon-metadata-repository/master/index.yaml && \
    colcon metadata update

# install ros2 packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-foxy-ros-base=0.9.2-1* \
    && rm -rf /var/lib/apt/lists/*

# add tros depend file
RUN echo "yaml https://raw.githubusercontent.com/HorizonRDK/trosdep/develop/foxy/trosdep.yaml" > \
    /etc/ros/rosdep/sources.list.d/50-tros.list

ENV DEB_BUILD_OPTIONS nocheck

ARG PLATFORM_RUN
ENV PLATFORM ${PLATFORM_RUN}

WORKDIR /tmp

ARG GIT_TOKEN
ARG USERNAME

RUN git clone -b master https://${USERNAME}:${GIT_TOKEN}@github.com/HorizonRDK/bloom.git \
    && cd bloom \
    && python3 setup.py install \
    && cd .. && rm -rf bloom

RUN git clone -b develop https://${USERNAME}:${GIT_TOKEN}@github.com/HorizonRDK/tros_release.git \
    && cd tros_release \
    && python3 setup.py install \
    && cd .. \
    && rm -rf tros_release

RUN apt-get update && apt-get install -q -y --no-install-recommends \
    curl \
    fakeroot \
    debhelper \
    dh-python

RUN curl -sSL http://sunrise.horizon.cc/keys/sunrise.gpg -o /usr/share/keyrings/sunrise.gpg \
    && echo "deb [arch=arm64 signed-by=/usr/share/keyrings/sunrise.gpg] http://sunrise.horizon.cc/ubuntu-rdk-beta focal main" > \
        /etc/apt/sources.list.d/sunrise.list

RUN apt-get update && apt-get install -q -y --no-install-recommends \
    hobot-multimedia-dev hobot-dnn libcjson-dev libasound2-dev python3-pip \
    libboost-dev libboost-system-dev libboost-serialization-dev \
    tros-ros-workspace

WORKDIR /root
