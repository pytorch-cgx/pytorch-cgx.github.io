---
layout: page
title: ""
permalink: /
---

# Welcome to Pytorch-CGX
[Pytorch-CGX](https://github.com/IST-DASLab/torch_cgx) is a pytorch extension adding a backend for pytorch distributed supporting allreduce of quantized buffers.
It supports quantizations of float16, float32 to 1-8 bits.

CGX is based on MPI torch.distributed backend. The extension essentially only replaces allreduce primitive.

# Install
```bash
export MPI_HOME=/path/to/mpi
export NCCL_HOME=/path/to/nccl
pip install pytorch-cgx
```

We also provide a [Dockerfile](https://github.com/IST-DASLab/torch_cgx/blob/master/Dockerfile) that allows to use `pytorch-cgx`.
See [install](/install) for the installation details.

# Introduction
`pytorch-cgx` is a plug-in extension for `torch.distributed` that substantially improves scalability on servers with low-bandwidth inter GPU communication.
Keep your training scripts the same, only need to replace `torch.distributed` backend and launching script.
The only changes to the training script using pytorch distributed required
are importing the built extension and specifying `cgx` as `torch.distributed.init_process_group` backend parameter and setting some environment variables.
See [usage](/usage) for the detailed instructions.

Example:
``` python
import torch
import torch.distributed as dist
import torch_cgx
# set `CGX_COMPRESSION_QUANTIZATION_BITS` to 4 to enable compression.

dist.init_process_group('cgx', init_method='env://', rank=args.local_rank)
```

Look at the speedup using CGX training on 8xRTX3090 compared to the Nvidia NCCL:

![Transformer-XL base](/assets/images/TXL_comparison.png){:class="img-responsive"}

# Cloud costs

We checked the prices of moving the training to the cloud.
We compared the low-bandwidth servers on Genesis Cloud with 8xRTX3090 gpus and bandwidth over-provisioned servers on AWS with 8 x V100.
As the benchmark we chose training of BERT on question-answering task

| Instance                  | Price per hour ($) | Training cost ($) | 
|---------------------------|--------------------|-------------------|
| Genesis Cloud without CGX | 10.4               | 61.7              | 
| AWS                       | 24.5               | 49.0              |
| Genesis Cloud with CGX    | 10.4               | **18.6**          |


See [benchmarks](/benchmarks) for further benchmark details.


# Citation
If you use this library in your research cite the corresponding paper as follows:
```tex
@inproceedings{markov2022cgx,
  title={CGX: Adaptive System Support for Communication-Efficient Deep Learning},
  author={Markov, Ilia and Ramezanikebrya, Hamidreza and Alistarh, Dan},
  booktitle={23rd International Middleware Conference (Middleware'22), November 7--11, 2022, Quebec, QC, Canada},
  year={2022}
}
```