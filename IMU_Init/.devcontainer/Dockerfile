FROM osrf/ros:noetic-desktop-full

# 定义用户和用户ID
ARG USERNAME=Nagisa
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ENV DEBIAN_FRONTEND=noninteractive

# 创建用户组（组名与用户名一致）
RUN groupadd --gid $USER_GID $USERNAME \
    # 创建用户并配置主组
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME \
    # # 配置密码（建议改用加密方式或删除密码配置）设置免密sudo后这句话没有必要
    # && echo "$USERNAME:password" | chpasswd \
    # 直接为用户配置免密 sudo（无需额外组）
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers \
    # 修复 .bashrc 所有权
    && chown $USERNAME:$USERNAME /home/$USERNAME/.bashrc

# 切换到 root 用户来执行系统软件包安装
USER root
# 一些必要软件的安装
RUN apt-get update \
    && apt-get install -y  net-tools nautilus bash-completion clangd git gedit gdb nano software-properties-common wget curl vim

# 添加 ROS 环境设置到 .bashrc
RUN echo "source /opt/ros/noetic/setup.bash" >> /home/${USERNAME}/.bashrc

# 创建工作空间并克隆代码库（最终修正版）
RUN apt-get update \
    # 安装系统依赖
    && apt-get install -y --no-install-recommends \
        libdw-dev \
        libelf-dev \
        cmake \
        libgoogle-glog-dev libgflags-dev \
        libatlas-base-dev \
        libeigen3-dev \
        libsuitesparse-dev

# 切换回 Nagisa 用户
USER $USERNAME

# 创建目标目录（ADD 会自动创建父目录，但显式声明更清晰）
RUN mkdir -p /home/Nagisa/ceres

# 复制并解压 tar 包到 /home/Nagisa/ceres
ADD ceres-solver-2.2.0.tar.gz /home/Nagisa/ceres/

# 所有后续操作在 /home/Nagisa/ceres 下执行
WORKDIR /home/Nagisa/ceres

# 编译安装
RUN mkdir ceres-bin && \
    cd ceres-bin && \
    cmake ../ceres-solver-2.2.0 && \
    make -j2 && \         
    make test 
# make test是使用开发者在修改代码后，通过 make test 快速验证改动是否引入新的错误
RUN cd ceres-bin && \
    sudo make install

# 清理编译中间文件（可选）
RUN rm -rf ceres-bin

# 创建工作空间并克隆代码
RUN mkdir -p /home/Nagisa/IMU_ws/src \
    && cd /home/Nagisa/IMU_ws/src
# 从宿主机复制代码到容器
COPY --chown=Nagisa:Nagisa code_utils /home/Nagisa/IMU_ws/src/code_utils
# COPY --chown=Nagisa:Nagisa imu_utils /home/Nagisa/IMU_ws/src/imu_utils

# 编译code_utils
# 在 Dockerfile 的编译步骤中分步操作
RUN cd /home/Nagisa/IMU_ws \
    && /bin/bash -c "source /opt/ros/noetic/setup.bash && catkin_make"

RUN cd /home/Nagisa/IMU_ws/src \
    && git config --global http.postBuffer 524288000 \
    && git config --global http.timeout 300 \
    && git clone --depth 1 --branch master https://github.com/mintar/imu_utils.git
# 编译imu_utils,byd这玩意必须要和前一个包分开编译，他依赖于前面的那个包的构建项
# 在 Dockerfile 的编译步骤中分步操作
RUN cd /home/Nagisa/IMU_ws \
    && /bin/bash -c "source /opt/ros/noetic/setup.bash && catkin_make"

# 设置工作目录
WORKDIR /home/$USERNAME

# 设置容器启动时默认执行的命令，并自动加载工作空间
CMD ["/bin/bash"]

