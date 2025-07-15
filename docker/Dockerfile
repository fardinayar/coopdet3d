# ───────────────────────────── 1. Base image ─────────────────────────────
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# ─────────────────────── 2. OS‑level dependencies ───────────────────────
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    tzdata build-essential g++ gcc wget git curl ca-certificates \
    lsb-release gnupg2 cmake ninja-build pkg-config \
    libcrypt-dev libgl1-mesa-glx libglib2.0-0 \
    libopenmpi-dev openmpi-bin python3-rosdep python3-rosinstall python3-vcstool && \
    rm -rf /var/lib/apt/lists/* && \
    ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    echo "Etc/UTC" > /etc/timezone

# ───────────────────── 3. ROS bootstrap ────────────────────────────────
RUN rosdep init && rosdep update

# ───────────────────── 4. CUDA / system include paths ──────────────────
ENV CUDA_HOME=/usr/local/cuda \
    PATH=/usr/local/cuda/bin:$PATH \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH \
    CPATH=/usr/local/cuda/include:/usr/include:/usr/include/x86_64-linux-gnu \
    C_INCLUDE_PATH=${CPATH} \
    CPLUS_INCLUDE_PATH=${CPATH}

# ───────────────────── 5. Miniconda & mamba ────────────────────────────
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && rm /tmp/miniconda.sh
ENV PATH=/opt/conda/bin:$PATH
RUN conda install -y -c conda-forge mamba && conda clean -afy

# ───────────────────── 6. Python env (py38) ────────────────────────────
RUN mamba create -y -n py38 python=3.8 && mamba clean -afy
SHELL ["conda", "run", "-n", "py38", "/bin/bash", "-c"]

#   6‑a. Use conda GCC so runtime & build libstdc++ match
RUN mamba install -y -c conda-forge gcc_linux-64=9.5.0 gxx_linux-64=9.5.0 libstdcxx-ng && mamba clean -afy

#   6‑b. Core DL stack (PyTorch 1.10.1 + CU113)
RUN pip install --no-cache-dir \
    torch==1.10.1+cu113 \
    torchvision==0.11.2+cu113 \
    torchaudio==0.10.1+cu113 \
    -f https://download.pytorch.org/whl/torch_stable.html

#   6‑c. MPI for torchpack
RUN mamba install -y -c conda-forge mpi4py=3.1.6 && mamba clean -afy

#   6‑d. Remaining Python deps for CoopDet3D + ROS tools
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

#   6‑e. Torchsparse quick installer
RUN curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | python - -v cu113

# ───────────────────── 7. Project workspace ────────────────────────────
WORKDIR /workspace
RUN git clone --depth 1 https://github.com/tum-traffic-dataset/coopdet3d.git
WORKDIR /workspace/coopdet3d

#   7‑a. Compile CUDA/C++ extensions (use matching arch list)
ENV TORCH_CUDA_ARCH_LIST="6.0;6.1;7.5;8.0+PTX"
RUN python setup.py develop

# ───────────────────── 8. Default shell QoL tweaks ─────────────────────
# Activate env & cd into repo on container start
RUN echo "source /opt/conda/etc/profile.d/conda.sh && conda activate py38 && cd /workspace/coopdet3d" >> /root/.bashrc
WORKDIR /workspace/coopdet3d
CMD ["/bin/bash"]
