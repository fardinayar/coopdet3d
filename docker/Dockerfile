# ───────────────────────── 1. Base image ─────────────────────────
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

# ─────────────────────── 2. OS-level deps ────────────────────────
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
        build-essential g++ gcc wget git curl ca-certificates \
        libgl1-mesa-glx libglib2.0-0 \
        libopenmpi-dev openmpi-bin && \
    ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    echo "Etc/UTC" > /etc/timezone && \
    rm -rf /var/lib/apt/lists/*

# ─────────────────────── 3. Miniconda + mamba ────────────────────
RUN wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && rm /tmp/miniconda.sh
ENV PATH=/opt/conda/bin:$PATH
RUN conda install -y -c conda-forge mamba && conda clean -afy

# ─────────────────────── 4. Conda env (Python 3.8) ───────────────
RUN mamba create -y -n py38 python=3.8 && mamba clean -afy
SHELL ["conda", "run", "-n", "py38", "/bin/bash", "-c"]

# ─────────────────────── 5. Core DL stack via pip ────────────────
# (pip wheels avoid the Intel-MKL 2024 symbol breakage)
RUN pip install --no-cache-dir \
        torch==1.10.1+cu113 torchvision==0.11.2+cu113 torchaudio==0.10.1 \
        -f https://download.pytorch.org/whl/torch_stable.html

# ─────────────────────── 6. mpi4py (pre-built) ───────────────────
RUN mamba install -y -c conda-forge mpi4py=3.1.6 && mamba clean -afy

# ─────────────────────── 7. Remaining Python deps ────────────────
RUN pip install --no-cache-dir \
        Pillow==8.4.0 tqdm torchpack \
        -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.10/index.html \
        mmcv-full==1.4.0 mmdet==2.20.0 \
        nuscenes-devkit numba==0.48.0 \
        seaborn thop catkin_pkg empy rospkg open3d

# ─────────────────────── 8. TorchSparse build ────────────────────
RUN curl -sL https://raw.githubusercontent.com/mit-han-lab/torchsparse/master/install.py | \
    python - --cuda 11.3

# ─────────────────────── 9. Finishing touches ────────────────────
ENV PYTHONUNBUFFERED=1
WORKDIR /workspace
CMD ["/bin/bash"]
