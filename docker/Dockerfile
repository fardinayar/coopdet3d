# ───────────────────────────── 1. Base image ─────────────────────────────
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# ─────────────────────── 2. OS‑level dependencies ───────────────────────
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends tzdata build-essential g++ gcc wget git curl ca-certificates lsb-release gnupg2 cmake ninja-build pkg-config libcrypt-dev libgl1-mesa-glx libglib2.0-0 libopenmpi-dev openmpi-bin && rm -rf /var/lib/apt/lists/* && ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime && echo "Etc/UTC" > /etc/timezone

# ─────────────────────── 3. CUDA environment paths ──────────────────────
ENV CUDA_HOME=/usr/local/cuda \
    PATH=/usr/local/cuda/bin:$PATH \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH \
    CPATH=/usr/local/cuda/include:/usr/include \
    C_INCLUDE_PATH=/usr/local/cuda/include:/usr/include \
    CPLUS_INCLUDE_PATH=/usr/local/cuda/include:/usr/include

# ─────────────────────── 4. ROS Noetic (desktop‑full) ────────────────────
ENV ROS_DISTRO=noetic
RUN curl -fsSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros-latest.list && \
    apt-get update && apt-get install -y --no-install-recommends python3-rosdep python3-rosinstall python3-rosinstall-generator python3-vcstool ros-noetic-desktop-full ros-noetic-uuid-msgs ros-noetic-vision-msgs ros-noetic-multisense-ros && \
    rm -rf /var/lib/apt/lists/*

RUN rosdep init && rosdep update

ENV ROS_ROOT=/opt/ros/${ROS_DISTRO} \
    PATH=/opt/ros/${ROS_DISTRO}/bin:$PATH \
    LD_LIBRARY_PATH=/opt/ros/${ROS_DISTRO}/lib:$LD_LIBRARY_PATH \
    PYTHONPATH=/opt/ros/${ROS_DISTRO}/lib/python3/dist-packages

# ─────────────────────── 5. Miniconda + mamba ────────────────────────────
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && rm /tmp/miniconda.sh
ENV PATH=/opt/conda/bin:$PATH
RUN conda install -y -c conda-forge mamba && conda clean -afy

# ─────────────────────── 6. Conda env (Python 3.8) ───────────────────────
RUN mamba create -y -n py38 python=3.8 && mamba clean -afy
SHELL ["conda", "run", "-n", "py38", "/bin/bash", "-c"]

# match GCC & libstdc++ inside env
RUN mamba install -y -c conda-forge gcc_linux-64=9.5.0 gxx_linux-64=9.5.0 libstdcxx-ng && mamba clean -afy

# ─────────────────────── 7. Core DL stack via pip ────────────────────────
RUN pip install --no-cache-dir torch==1.10.1+cu113 torchvision==0.11.2+cu113 torchaudio==0.10.1 -f https://download.pytorch.org/whl/torch_stable.html

# ─────────────────────── 8. mpi4py & misc deps ───────────────────────────
RUN mamba install -y -c conda-forge mpi4py=3.1.6 && mamba clean -afy
RUN pip install --no-cache-dir \
    Pillow==8.4.0 \
    tqdm \
    torchpack \
    -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
    mmcv-full==1.4.0 \
    mmdet==2.20.0 \
    nuscenes-devkit \
    numba==0.48.0 \
    seaborn \
    thop \
    catkin_pkg \
    empy \
    rospkg \
    open3d

# ─────────────────────── 9. TorchSparse build ────────────────────────────
RUN curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | python - --cuda 11.3

# ────────────────────── 10. Clone & install coopdet3d ────────────────────
WORKDIR /workspace
RUN git clone --depth 1 https://github.com/tum-traffic-dataset/coopdet3d.git
WORKDIR /workspace/coopdet3d
ENV TORCH_CUDA_ARCH_LIST="6.0;6.1;7.5;8.0+PTX"
RUN python setup.py develop

# ────────────────────── 11. QoL shell tweaks ─────────────────────────────
RUN echo 'source /opt/conda/etc/profile.d/conda.sh && conda activate py38' >> /etc/bash.bashrc && \
    echo 'source /opt/ros/${ROS_DISTRO}/setup.bash' >> /etc/bash.bashrc && \
    echo 'cd /workspace/coopdet3d' >> /etc/bash.bashrc

# ────────────────────── 12. Entrypoint ──────────────────────────────────
ENV PYTHONUNBUFFERED=1
WORKDIR /workspace/coopdet3d
CMD ["/bin/bash"]
