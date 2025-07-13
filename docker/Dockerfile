FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# Avoid interactive prompts
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# ───── System dependencies ─────
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential g++ gcc wget git curl \
    libgl1-mesa-glx libglib2.0-0 \
    openmpi-bin openmpi-common libopenmpi-dev libgtk2.0-dev \
    && rm -rf /var/lib/apt/lists/*

# ───── Miniconda + Mamba ─────
ENV CONDA_DIR=/opt/conda
ENV PATH=$CONDA_DIR/bin:$PATH

RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p $CONDA_DIR && \
    rm /tmp/miniconda.sh && \
    conda install -y -n base -c conda-forge mamba && \
    conda clean -afy

# ───── Create Python 3.8 env with PyTorch 1.10.1 ─────
RUN mamba create -y -n py38 python=3.8 \
    pytorch=1.10.1 torchvision=0.11.2 torchaudio=0.10.1 \
    cudatoolkit=11.3 -c pytorch -c conda-forge && \
    mamba clean -afy

# Set SHELL so all following RUNs happen inside conda env
SHELL ["conda", "run", "--no-capture-output", "-n", "py38", "/bin/bash", "-c"]

# ───── Add Intel runtimes to support PyTorch → fix iJIT_NotifyEvent ─────
RUN mamba install -y intel-openmp ittapi -c conda-forge && \
    mamba clean -afy

# ───── Install common Python packages ─────
RUN pip install --no-cache-dir \
    Pillow==8.4.0 tqdm torchpack \
    -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
    mmcv-full==1.4.0 mmdet==2.20.0 \
    nuscenes-devkit mpi4py==3.0.3 numba==0.48.0 \
    seaborn thop catkin_pkg empy rospkg open3d

# ───── TorchSparse install (requires functional torch import) ─────
RUN export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH && \
    curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3

# ───── Optional: cleanup ─────
RUN conda clean -afy && pip cache purge
