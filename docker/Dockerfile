FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# ───────────────── system deps ──────────────────
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# system packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential g++ gcc wget git curl \
        libgl1-mesa-glx libglib2.0-0 \
        openmpi-bin openmpi-common libopenmpi-dev libgtk2.0-dev \
        tzdata && \
    rm -rf /var/lib/apt/lists/*

# ───────────────── Miniconda + Mamba ────────────
ENV CONDA_DIR=/opt/conda
ENV PATH=$CONDA_DIR/bin:$PATH

RUN wget -qO /tmp/miniconda.sh \
        https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p $CONDA_DIR && rm /tmp/miniconda.sh && \
    conda install -y -n base -c conda-forge mamba && \
    conda clean -afy

# ───────────────── build our env in ONE shot ───
RUN mamba create -y -n py38 python=3.8 \
        pytorch=1.10.1 torchvision=0.11.2 torchaudio=0.10.1 \
        cudatoolkit=11.3 -c pytorch -c conda-forge && \
    mamba clean -afy

# Make every subsequent RUN execute inside py38
SHELL ["conda", "run", "--no-capture-output", "-n", "py38", "/bin/bash", "-c"]

# ───────────────── python-level packages ────────
# Step 1 — Install PyTorch and dependencies
RUN mamba install -y -n py38 intel-openmp && mamba clean -afy

# Step 2 — Install all pip packages
RUN pip install --no-cache-dir \
    Pillow==8.4.0 tqdm torchpack \
    -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
    mmcv-full==1.4.0 mmdet==2.20.0 \
    nuscenes-devkit mpi4py==3.0.3 numba==0.48.0 \
    seaborn thop catkin_pkg empy rospkg open3d

# Step 3 — Only now: run the TorchSparse installer
RUN conda run -n py38 /bin/bash -c "\
    export LD_LIBRARY_PATH=/opt/conda/envs/py38/lib:\$LD_LIBRARY_PATH && \
    curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3"



# ───────────────── ROS bits (unchanged) ─────────
RUN apt-get update && apt-get install -y --no-install-recommends \
        cmake gnupg2 lsb-release ca-certificates && \
    echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" \
        > /etc/apt/sources.list.d/ros-latest.list && \
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add - && \
    apt-get update && apt-get install -y --no-install-recommends \
        zip htop screen libgl1-mesa-glx libpython3-dev \
        python3-rosdep python3-rosinstall python3-rosinstall-generator python3-vcstool \
        ros-noetic-ros-base ros-noetic-multisense-ros ros-noetic-uuid-msgs ros-noetic-vision-msgs && \
    rm -rf /var/lib/apt/lists/*
