ARG BASE_IMAGE_NAME="nvidia/cuda"
ARG BASE_IMAGE_TAG="12.8.0-devel-ubi9"
FROM $BASE_IMAGE_NAME:$BASE_IMAGE_TAG

# Install tools
#
# clang-tools-extra for clang-format
# gcc-toolset-13 compiler
# vim-enhanced for color vi support in git
# zip/unzip for vcpkg
#
RUN dnf update -y \
  && dnf -y install \
  clang-tools-extra \
  gcc-toolset-13 \
  git \
  sudo \
  unzip \
  vim-enhanced \
  zip \
  && dnf clean all \
  && rm -rf /var/cache/yum

# Install cmake
WORKDIR /opt
RUN curl --fail -LO "https://github.com/Kitware/CMake/releases/download/v3.31.6/cmake-3.31.6-linux-x86_64.tar.gz" \
  && tar -xvf cmake-3.31.6-linux-x86_64.tar.gz \
  && rm cmake-3.31.6-linux-x86_64.tar.gz

# Install ninja
WORKDIR /opt/ninja-1.12.1
RUN curl --fail -LO "https://github.com/ninja-build/ninja/releases/download/v1.12.1/ninja-linux.zip" \
  && unzip ninja-linux.zip \
  && rm ninja-linux.zip

# 🧑‍💻 Create the developer (non-root) user
ARG USERNAME
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user and group
RUN groupadd --gid $USER_GID $USERNAME \
  && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
  && echo $USERNAME ALL=\(ALL\) NOPASSWD: ALL > /etc/sudoers.d/$USER_UID \
  && chmod 0440 /etc/sudoers.d/$USER_UID

# 🧑‍💻 Change to non-root user
USER $USERNAME

# ⭕ Install vcpkg. Clone & bootstrap
#
# Install to user $HOME folder since / requires root permissions.
#
WORKDIR /home/$USERNAME
RUN git clone https://github.com/microsoft/vcpkg.git \
&& ./vcpkg/bootstrap-vcpkg.sh -disableMetrics

# Install bashrc configuration
WORKDIR /home/$USERNAME/.bashrc.d
COPY --chown=$USER_UID:$USER_GID git.bashrc .
RUN printf "alias ls='ls --color'\n" > alias.bashrc

# 🌊 Setup the container environment

# Add gcc to path
ENV PATH="/opt/rh/gcc-toolset-13/root/usr/bin:$PATH"

# Add cmake to path
ENV CMAKE_HOME="/opt/cmake-3.31.6-linux-x86_64"
ENV PATH="$CMAKE_HOME/bin:$PATH"

# Add ninja to path
ENV PATH="/opt/ninja-1.12.1:$PATH"

# Set vcpkg root to install folder
ENV VCPKG_ROOT="/home/$USERNAME/vcpkg"
ENV PATH="$VCPKG_ROOT:$PATH"

CMD [ "bash" ]
