FROM ros:kinetic-ros-core

ARG NB_USER="val"
ARG NB_UID="1000"
ARG NB_GID="100"

USER root

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
    git git-gui dump openssl expect libssh-dev nano gedit unzip \
    openjdk-8-jre openjfx python-catkin-tools mesa-utils x11-xserver-utils \
    ros-kinetic-gazebo-ros ros-kinetic-gazebo-ros ros-kinetic-robot-state-publisher \
    wget bzip2 ca-certificates sudo locales fonts-liberation && \
    rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen && \
    sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Configure environment
ENV SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV HOME=/home/$NB_USER

# Add a script that we will use to correct permissions after running certain commands
ADD scripts/fix-permissions /usr/local/bin/fix-permissions

# Create NB_USER wtih name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    /bin/bash -c 'echo -e "password\npassword" | passwd $NB_USER' && \
    usermod -aG adm,sudo,dialout,plugdev val && \
    fix-permissions /home/$NB_USER 

USER $NB_UID
RUN /bin/bash -c ". /opt/ros/kinetic/setup.bash && rosdep update"
RUN mkdir -p /home/$NB_USER/catkin_ws/src
RUN echo "source /opt/ros/kinetic/setup.bash" >> /home/$NB_USER/.bashrc && \
    cd /home/$NB_USER/catkin_ws/ && \
    /bin/bash -c ". /opt/ros/kinetic/setup.bash && catkin init && catkin config --install" && cd src && \
    git clone https://github.com/ihmcrobotics/ihmc_ros_core.git --depth 1 --branch release/0.10.0 && \
    git clone https://github.com/openhumanoids/ihmc_valkyrie_ros.git --depth 1 --branch vi-local-0.10.0 && \
    # cannot shallow clone val_description due to setup.py
    git clone https://github.com/ipab-slmc/val_description.git --branch uoe_december_2019 && \ 
    git clone https://github.com/ihmcrobotics/ihmc-ros-control.git --depth 1 --branch develop

USER root
RUN apt-get update && /bin/bash -c ". /opt/ros/kinetic/setup.bash && rosdep install --from-path /home/$NB_USER/catkin_ws/src -iy" && \
    rm -rf /var/lib/apt/lists/*

USER $NB_UID
RUN cd /home/$NB_USER/catkin_ws && \
    /bin/bash -c ". /opt/ros/kinetic/setup.bash && catkin config --install && catkin build" && \
    rm -rf /home/$NB_USER/catkin_ws/build && \
    echo "source /home/$NB_USER/catkin_ws/install/setup.bash" >> /home/$NB_USER/.bashrc

RUN rm -rf /home/$NB_USER/catkin_ws/devel /home/$NB_USER/catkin_ws/build /home/$NB_USER/catkin_ws/logs

# TODO: Generate these with a gradle script
RUN wget https://terminator.robots.inf.ed.ac.uk/public/maven_local.zip -q -O /home/$NB_USER/maven_local.zip && \
    unzip -d /home/$NB_USER /home/$NB_USER/maven_local.zip && \
    rm /home/$NB_USER/maven_local.zip
RUN fix-permissions /home/$NB_USER

RUN cd /home/$NB_USER/catkin_ws && \
    /bin/bash -c ". /opt/ros/kinetic/setup.bash && . /home/$NB_USER/catkin_ws/install/setup.bash && roslaunch ihmc_valkyrie_ros valkyrie_warmup_gradle_cache.launch"

RUN mkdir -p /home/$NB_USER/.ihmc
ADD scripts/IHMCNetworkParameters.ini /home/$NB_USER/.ihmc

USER root
WORKDIR /home/$NB_USER

ENV SHELL=/bin/bash
EXPOSE 11311

USER $NB_UID