ARG BASE_DOCKER_IMAGE=osrf/ros:humble-desktop-full
FROM $BASE_DOCKER_IMAGE AS base

SHELL ["/bin/bash", "-c"]

ENV DEBIAN_FRONTEND=noninteractive

# Create a non-root user if necessary

ARG USERNAME=ros
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# I assume there is already a user created in the base image
#ARG USERNAME=ubuntu
#ENV USERNAME=$USERNAME

RUN groupadd --gid $USER_GID $USERNAME \
  && useradd -m \
  -s /bin/bash \
  -G sudo,plugdev,dialout \
  --uid $USER_UID \
  --gid $USER_GID \
  $USERNAME

RUN apt update \
    && apt install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

# Basic tools
RUN apt update && apt install -y \
    python3-pip \
    python3-argcomplete \
    python3-colcon-* \
    btop \
    bat \
    git \
    vim \
    nano \
    gdb \
    rsync \
    strace \
    ros-dev-tools \
    bash-completion \
    fzf \
    zsh \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Install nano syntax highlighting
RUN curl https://raw.githubusercontent.com/scopatz/nanorc/master/install.sh | sh

ENV EDITOR=nano

USER ${USERNAME}

# Set up autocompletion for user
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /home/$USERNAME/.bashrc \
    && echo "if [ -f /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash ]; then source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash; fi" >> /home/$USERNAME/.bashrc \
    && echo "if [ -f /usr/share/colcon_cd/function/colcon_cd.sh ]; then source /usr/share/colcon_cd/function/colcon_cd.sh; export _colcon_cd_root=/opt/ros/${ROS_DISTRO}; fi" >> /home/$USERNAME/.bashrc \
    && echo "set show-all-if-ambiguous on" >> /home/$USERNAME/.bashrc \
    && echo "set completion-ignore-case on" >> /home/$USERNAME/.bashrc \
    && echo "bind 'TAB:menu-complete'" >> /home/$USERNAME/.bashrc


# Set up zoxide for smarter cd
RUN curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh \
    && echo "eval $(zoxide init bash --cmd cd)" >> /home/$USERNAME/.bashrc

# Setup bash aliases
RUN echo "alias ll='ls -alF'" >> /home/$USERNAME/.bashrc \
    && echo "alias la='ls -A'" >> /home/$USERNAME/.bashrc \
    && echo "alias l='ls -CF'" >> /home/$USERNAME/.bashrc \
    && echo "alias clc='clear'" >> /home/$USERNAME/.bashrc \
    && echo "alias ..='cd ..'" >> /home/$USERNAME/.bashrc \
    && echo "alias fzf='fzf --preview \"bat --color=always --line-range :500 {}\"'" >> /home/$USERNAME/.bashrc \
    #&& echo "alias fzf='fzf --bind 'enter:become(nano {})'" >> /home/$USERNAME/.bashrc \
    && echo "alias reload='source ~/.bashrc'" >> /home/$USERNAME/.bashrc


ENV DEBIAN_FRONTEND=

FROM base AS app

ENV DEBIAN_FRONTEND=noninteractive

# Install ROS 2 big packages
RUN sudo apt update && sudo apt install -y \ 
    ros-${ROS_DISTRO}-plotjuggler-ros \
    ros-${ROS_DISTRO}-rqt* \
    ros-${ROS_DISTRO}-ament-* \
    ros-${ROS_DISTRO}-rviz2 \
    ros-${ROS_DISTRO}-navigation2 \
    ros-${ROS_DISTRO}-nav2-bringup \
    ros-${ROS_DISTRO}-rtabmap-ros

# Install other software
#RUN sudo apt update && sudo apt install -y \

ARG WORKSPACE=/ros_ws
ENV WORKSPACE=$WORKSPACE
WORKDIR $WORKSPACE

RUN echo "if [ -f ${WORKSPACE}/install/setup.bash ]; then source ${WORKSPACE}/install/setup.bash; fi" >> /home/$USERNAME/.bashrc

ENV DEBIAN_FRONTEND=