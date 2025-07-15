# ───────────────────────── 1. Base image ─────────────────────────
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# ─────────────────── 2. OS-level dependencies ────────────────────
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
        # build & system utils
        tzdata build-essential g++ gcc wget git curl ca-certificates \
        lsb-release gnupg2 cmake ninja-build pkg-config \
        # libs often needed by Torch / Open3D / ROS
        libcrypt-dev libgl1-mesa-glx libglib2.0-0 \
        libopenmpi-dev openmpi-bin \
        # lightweight ROS bootstrap tools
        python3-rosdep python3-rosinstall \
    && rm -rf /var/lib/apt/lists/* \
    && ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime \
    && echo "Etc/UTC" > /etc/timezone

# ROS1 GPG key (for rosdep)
RUN curl -fsSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | \
    gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg

# rosdep database (tolerate re-runs)
RUN rosdep init || echo "rosdep already initialized" && rosdep update

# ─────────────── 3. Miniconda + mamba stack ───────────────
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh && \
    /opt/conda/bin/conda clean -afy
ENV PATH=/opt/conda/bin:$PATH

RUN conda install -y -c conda-forge mamba && conda clean -afy
RUN mamba create -y -n py38 python=3.8 && mamba clean -afy

# Every following RUN executes inside the conda env
SHELL ["conda", "run", "-n", "py38", "/bin/bash", "-c"]

# ─────────────── 4. Compilers + PyTorch GPU ───────────────
RUN mamba install -y -c conda-forge gcc_linux-64=9.5.0 gxx_linux-64=9.5.0 libstdcxx-ng && mamba clean -afy

# PyTorch 1.10.1 CUDA-11.3 wheels (extra index)
RUN pip install --upgrade pip && \
    pip install --no-cache-dir \
        --extra-index-url https://download.pytorch.org/whl/cu113 \
        torch==1.10.1+cu113 \
        torchvision==0.11.2+cu113 \
        torchaudio==0.10.1+cu113

# ─────────────── 5. Other Python deps ───────────────
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
        vcstool \
        open3d

# torchsparse
RUN curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | python -

# ─────────────── 6. Build env tweaks ───────────────
ENV CUDA_HOME=/usr/local/cuda \
    CPATH=/usr/include/x86_64-linux-gnu:/usr/local/cuda/include:$CPATH \
    CPLUS_INCLUDE_PATH=$CPATH \
    TORCH_CUDA_ARCH_LIST="6.0;6.1;7.5;8.0+PTX"

# ─────────────── 7. CoopDet3D checkout & build ───────────────
WORKDIR /workspace
RUN git clone --depth 1 https://github.com/tum-traffic-dataset/coopdet3d.git
WORKDIR /workspace/coopdet3d
RUN python setup.py develop

# ─────────────── 8. QoL shell tweaks ───────────────
# auto-activate env and land in repo
RUN echo 'conda activate py38' >> /etc/bash.bashrc && \
    echo 'cd /workspace/coopdet3d' >> /etc/bash.bashrc
ENV PATH=/opt/conda/envs/py38/bin:$PATH
WORKDIR /workspace/coopdet3d

# Default to an interactive shell
CMD ["/bin/bash"]
