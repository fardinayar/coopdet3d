# Base image
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# Step 1 — System dependencies
# Avoid timezone prompt
ENV DEBIAN_FRONTEND=noninteractive
RUN ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential g++ gcc wget git curl ca-certificates \
        libgl1-mesa-glx libglib2.0-0 libopenmpi-dev openmpi-bin && \
    rm -rf /var/lib/apt/lists/*

# Step 2 — Install Miniconda + Mamba
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && rm /tmp/miniconda.sh
ENV PATH=/opt/conda/bin:$PATH

# Step 3 — Create PyTorch Conda environment
RUN conda install -y -c conda-forge mamba && \
    mamba create -y -n py38 python=3.8 \
        pytorch=1.10.1 torchvision=0.11.2 torchaudio=0.10.1 \
        cudatoolkit=11.3 -c pytorch -c conda-forge && \
    conda clean -afy

# Step 4 — Intel OpenMP support
RUN mamba install -y -n py38 intel-openmp ittapi -c conda-forge && \
    mamba clean -afy

# Step 5 — Set environment variables
ENV CONDA_DEFAULT_ENV=py38
ENV PATH="/opt/conda/envs/py38/bin:$PATH"
ENV LD_LIBRARY_PATH="/opt/conda/envs/py38/lib:/usr/local/nvidia/lib:/usr/local/nvidia/lib64:$LD_LIBRARY_PATH"

# Step 6 — Install pip packages inside Conda env
RUN conda run -n py38 pip install --no-cache-dir \
    Pillow==8.4.0 tqdm torchpack \
    -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
    mmcv-full==1.4.0 mmdet==2.20.0 \
    nuscenes-devkit mpi4py numba==0.48.0 \
    seaborn thop catkin_pkg empy rospkg open3d

# Step 7 — Install TorchSparse
RUN conda run -n py38 /bin/bash -c "\
    curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3"

# Default shell
SHELL ["/bin/bash", "-c"]
