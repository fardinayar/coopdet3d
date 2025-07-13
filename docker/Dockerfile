FROM nvidia/cuda:11.3.1-devel-ubuntu20.04
ENV DEBIAN_FRONTEND=noninteractive

# ─── system deps ───
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential g++ gcc wget git curl \
        libgl1-mesa-glx libglib2.0-0 ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# ─── miniconda + mamba ───
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && rm /tmp/miniconda.sh
ENV PATH=/opt/conda/bin:$PATH

# ─── create env with torch 1.10/cu113 ───
RUN conda install -y -c conda-forge mamba && \
    mamba create -y -n py38 python=3.8 \
        pytorch=1.10.1 torchvision=0.11.2 torchaudio=0.10.1 \
        cudatoolkit=11.3 -c pytorch -c conda-forge && \
    conda clean -afy

# ─── Intel OpenMP + ITT runtime ───
RUN mamba install -y -n py38 intel-openmp ittapi -c conda-forge && \
    mamba clean -afy
ENV LD_LIBRARY_PATH=/opt/conda/envs/py38/lib:$LD_LIBRARY_PATH

# ─── mpi4py from conda (pre-built) ───                                           # NEW
RUN mamba install -y -n py38 mpi4py=3.1.6 -c conda-forge && mamba clean -afy      # NEW

# ─── pip packages ───
RUN conda run -n py38 pip install --no-cache-dir \
    Pillow==8.4.0 tqdm torchpack \
    -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
    mmcv-full==1.4.0 mmdet==2.20.0 \
    nuscenes-devkit numba==0.48.0 \
    seaborn thop catkin_pkg empy rospkg open3d

# ─── TorchSparse ───
RUN conda run -n py38 /bin/bash -c "\
    curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3"

# default env
SHELL ["/bin/bash", "-c"]
ENV PATH=/opt/conda/envs/py38/bin:$PATH
