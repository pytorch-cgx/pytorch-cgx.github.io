---
layout: page
title: Install
permalink: /install
---

# Prerequisites
CGX, as a pytorch extension, requires `pytorch>=1.10.0`.

For faster build we recommend to have `ninja` installed (`pip install ninja`).

The compression is only supported for GPU-based buffers so either CUDA or ROCm is required.
If CUDA or ROCm are installed not in the standard paths, set `[CUDA|ROCM]_HOME` or `[CUDA|ROCM]_PATH` accordingly.

As long as it is based on MPI, it requires OpenMPI with GPU support installed (other MPI implementations were not tested).
Also, the library supports NCCL based communications, so it requires NVIDIA NCCL library.


Set `MPI_HOME` environment variable to mpi home. In case of AMD GPU, set `CGX_CUDA` to 0.
Set `NCCL_HOME` environment variable to NCCL home, or `NCCL_INCLUDE` and `NCCL_LIB`.
Set `QSGD_DETERMENISTIC=0` if you want to have stochastic version QSGD.

## Download
```bash
pip install pytorch_cgx
```

## Build from source
```bash
git clone https://github.com/IST-DASLab/torch_cgx
python setup.py install
```
