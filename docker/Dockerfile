# Base image with CUDA 11.3 and Ubuntu 20.04
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# Step 1 — System dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential g++ gcc wget git curl \
    libgl1-mesa-glx libglib2.0-0 ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Step 2 — Install Miniconda and Mamba
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh
ENV PATH="/opt/conda/bin:$PATH"

# Step 3 — Create Conda environment with PyTorch 1.10.1 + CUDA 11.3
RUN conda install -y -c conda-forge mamba && \
    mamba create -y -n py38 python=3.8 \
        pytorch=1.10.1 torchvision=0.11.2 torchaudio=0.10.1 \
        cudatoolkit=11.3 -c pytorch -c conda-forge && \
    conda clean -afy

# Step 4 — Intel OpenMP + ITT runtime (needed for PyTorch loading)
RUN mamba install -y -n py38 intel-openmp ittapi -c conda-forge && \
    mamba clean -afy

# Tell the dynamic linker where to find those shared libs
ENV LD_LIBRARY_PATH=/opt/conda/envs/py38/lib:$LD_LIBRARY_PATH

# Step 5 — Install Python packages with pip
RUN conda run -n py38 pip install --no-cache-dir \
    Pillow==8.4.0 tqdm torchpack \
    -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
    mmcv-full==1.4.0 mmdet==2.20.0 \
    nuscenes-devkit mpi4py==3.1.6 numba==0.48.0 \
    seaborn thop catkin_pkg empy rospkg open3d

# Step 6 — Install TorchSparse (must run after torch is working)
RUN conda run -n py38 /bin/bash -c "\
    curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3"

# Activate conda env by default
SHELL ["/bin/bash", "-c"]
ENV CONDA_DEFAULT_ENV=py38
ENV PATH="/opt/conda/envs/py38/bin:$PATH"
