FROM nvidia/cuda:11.3.1-devel-ubuntu20.04
ENV DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC

# ───────── system deps ─────────
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential g++ gcc wget git curl \
        libgl1-mesa-glx libglib2.0-0 \
        openmpi-bin libopenmpi-dev && \
    rm -rf /var/lib/apt/lists/*

# ───────── miniconda + mamba ─────────
ENV CONDA_DIR=/opt/conda
ENV PATH=$CONDA_DIR/bin:$PATH
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p $CONDA_DIR && rm /tmp/miniconda.sh && \
    conda install -y -n base -c conda-forge mamba && \
    conda clean -afy

# ───────── create env with PyTorch 1.10.1 (CU11.3) ─────────
RUN mamba create -y -n py38 python=3.8 \
        pytorch=1.10.1 torchvision=0.11.2 torchaudio=0.10.1 \
        cudatoolkit=11.3 -c pytorch -c conda-forge && \
    mamba clean -afy

# default shell executes inside the env
SHELL ["conda", "run", "--no-capture-output", "-n", "py38", "/bin/bash", "-c"]

# ───────── Intel OpenMP + ITT runtime ─────────
RUN mamba install -y intel-openmp ittapi -c conda-forge && mamba clean -afy

# make sure the linker can always see both libs
ENV LD_PRELOAD="$CONDA_PREFIX/lib/libiomp5.so:$CONDA_PREFIX/lib/libittnotify.so"

# ───────── python-level packages ─────────
RUN pip install --no-cache-dir \
      Pillow==8.4.0 tqdm torchpack \
      -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
      mmcv-full==1.4.0 mmdet==2.20.0 \
      nuscenes-devkit mpi4py==3.0.3 numba==0.48.0 \
      seaborn thop catkin_pkg empy rospkg open3d

# ───────── TorchSparse install ─────────
RUN curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3

# ───────── clean up ─────────
RUN conda clean -afy && pip cache purge
