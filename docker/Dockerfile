ARG BASE_DOCKER_IMAGE=osrf/ros:humble-desktop-full
FROM $BASE_DOCKER_IMAGE AS base

SHELL ["/bin/bash", "-c"]

ENV DEBIAN_FRONTEND=noninteractive

# Create a non-root user

ARG USERNAME=ros
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
  && useradd -m \
  -s /bin/bash \
  -G sudo,plugdev,dialout \
  --uid $USER_UID \
  --gid $USER_GID \
  $USERNAME

RUN apt update && apt full-upgrade -y \
    && apt install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

# Basic tools
RUN apt update && apt install -y \
    apt-utils \
    python3-pip \
    python3-argcomplete \
    python3-colcon-* \
    btop \
    bat \
    git \
    vim \
    nano \
    gdb \
    gawk \
    rsync \
    strace \
    ros-dev-tools \
    bash-completion \
    zsh \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Install nano syntax highlighting
RUN curl https://raw.githubusercontent.com/scopatz/nanorc/master/install.sh | sh

ENV EDITOR=nano

USER ${USERNAME}
ENV HOME /home/${USERNAME}
ENV USER ${USERNAME}
ENV PATH=${HOME}/.local/bin:${PATH}

# Set up autocompletion for user
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ${HOME}/.bashrc \
    && echo "if [ -f /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash ]; then source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash; fi" >> ${HOME}/.bashrc \
    && echo "if [ -f /usr/share/colcon_cd/function/colcon_cd.sh ]; then source /usr/share/colcon_cd/function/colcon_cd.sh; export _colcon_cd_root=/opt/ros/${ROS_DISTRO}; fi" >> ${HOME}/.bashrc \
    && sed -i 's/#force_color_prompt=yes/force_color_prompt=yes/' ${HOME}/.bashrc 

# Install ble:sh, Set up zoxide for smarter cd and fzf for fuzzy search and lsd for ls
RUN git clone --recursive --depth 1 --shallow-submodules https://github.com/akinomyoga/ble.sh.git ${HOME}/ble.sh.d \
    && wget https://github.com/lsd-rs/lsd/releases/download/v1.1.5/lsd-musl_1.1.5_amd64.deb -O ${HOME}/lsd.deb \
    && sudo dpkg -i ${HOME}/lsd.deb && rm ${HOME}/lsd.deb \
    && make -j -C ${HOME}/ble.sh.d install PREFIX=${HOME}/.local \
    && curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh \
    && git clone --depth 1 https://github.com/junegunn/fzf.git ${HOME}/.fzf && ${HOME}/.fzf/install \
    && curl -sS https://starship.rs/install.sh | sh -s -- -y -b ${HOME}/.local/bin \
    && echo 'eval "$(fzf --bash)"' >> ${HOME}/.bashrc \
    && sed -i '1isource ${HOME}/.local/share/blesh/ble.sh --noattach' ${HOME}/.bashrc \
    && echo 'eval "$(zoxide init bash --cmd cd)"' >> ${HOME}/.bashrc \
    && echo 'eval "$(starship init bash)"' >> ${HOME}/.bashrc \
    && echo '[[ ${BLE_VERSION-} ]] && ble-attach' >> ${HOME}/.bashrc \
    && mkdir -p ${HOME}/.config \
    && touch ${HOME}/.config/starship.toml \
    && touch ${HOME}/.blerc \
    && starship preset gruvbox-rainbow >> ${HOME}/.config/starship.toml \
    && echo 'source /etc/profile.d/bash_completion.sh' >> ${HOME}/.blerc \
    && echo 'ble-import -d integration/fzf-completion' >> ${HOME}/.blerc \
    && echo 'ble-import -d integration/fzf-key-bindings' >> ${HOME}/.blerc \
    && echo 'ble-import integration/zoxide' >> ${HOME}/.blerc

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
    ros-${ROS_DISTRO}-ros-gz
    #ros-${ROS_DISTRO}-rtabmap-ros

# Install other software
#RUN sudo apt update && sudo apt install -y \

# Clean up
RUN sudo apt autoremove -y \
    && sudo apt clean \
    && sudo rm -rf /var/lib/apt/lists/*

ARG WORKSPACE=/ros_ws
ENV WORKSPACE=$WORKSPACE
WORKDIR $WORKSPACE

RUN echo "if [ -f ${WORKSPACE}/install/setup.bash ]; then source ${WORKSPACE}/install/setup.bash; fi" >> ${HOME}/.bashrc

ENV DEBIAN_FRONTEND=