# ───────────────────────────── 1. Base image ─────────────────────────────
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# ─────────────────────── 2. OS‑level dependencies ───────────────────────
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    # base build utils + misc
    tzdata build-essential g++ gcc wget git curl ca-certificates \
    lsb-release gnupg2 cmake ninja-build pkg-config \
    # libs needed by Torch / Open3D / ROS
    libcrypt-dev libgl1-mesa-glx libglib2.0-0 \
    libopenmpi-dev openmpi-bin \
    # lightweight ROS bootstrap tools
    python3-rosdep python3-rosinstall && \
    rm -rf /var/lib/apt/lists/* && \
    ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime && echo "Etc/UTC" > /etc/timezone

# ROS1 GPG key (for rosdep) ────────────────────────────────────────────────
RUN curl -fsSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | \
    gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg

# Initialise rosdep (no ROS distro packages are actually installed here) ----
RUN rosdep init && rosdep update

# ───────────────────── 3. Miniconda & mamba stack ─────────────────────────
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && rm /tmp/miniconda.sh && \
    /opt/conda/bin/conda install -y -c conda-forge mamba && /opt/conda/bin/conda clean -afy
ENV PATH="/opt/conda/bin:$PATH"

# Python 3.8 env (torch 1.10 + cu113 requires ≤3.9) --------------------------
RUN mamba create -y -n py38 python=3.8 && mamba clean -afy

# System GCC inside the env needs to see libstdc++=9.5.0 for ABI compat ------
RUN mamba install -y -n py38 -c conda-forge gcc_linux-64=9.5.0 gxx_linux-64=9.5.0 libstdcxx-ng && mamba clean -afy

# ─────────────── 4. Deep‑learning / vision Python deps ─────────────────────
RUN pip install --no-cache-dir \
    torch==1.10.1+cu113 \
    torchvision==0.11.2+cu113 \
    torchaudio==0.10.1+cu113 \
    -f https://download.pytorch.org/whl/cu113/torch_stable.html

RUN mamba install -y -n py38 -c conda-forge mpi4py=3.1.6 && mamba clean -afy

# mmcv / mmdet etc.  ---------------------------------------------------------
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
    open3d \
    vcstool

# TorchSparse install script (already includes its own C++/CUDA build) -------
RUN curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - -v 0.6.0 --cuda 11.3

# ───────────────────── 5. Environment paths for CUDA / glibc ───────────────
ENV CUDA_HOME=/usr/local/cuda \
    PATH=/usr/local/cuda/bin:$PATH \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH \
    CPATH=/usr/local/cuda/include:/usr/include:/usr/include/x86_64-linux-gnu \
    C_INCLUDE_PATH=/usr/local/cuda/include:/usr/include:/usr/include/x86_64-linux-gnu \
    CPLUS_INCLUDE_PATH=/usr/local/cuda/include:/usr/include:/usr/include/x86_64-linux-gnu \
    TORCH_CUDA_ARCH_LIST="6.0;6.1;7.5;8.0+PTX"

# ───────────────────────── 6. Clone & build CoopDet3D ───────────────────────
WORKDIR /workspace
RUN git clone --depth 1 https://github.com/tum-traffic-dataset/coopdet3d.git
WORKDIR /workspace/coopdet3d
RUN python setup.py develop

# ───────────────────── 7. QoL shell tweaks (auto‑activate env) ──────────────
RUN echo "source /opt/conda/etc/profile.d/conda.sh && conda activate py38" >> /etc/bash.bashrc
WORKDIR /workspace/coopdet3d

CMD ["bash"]
