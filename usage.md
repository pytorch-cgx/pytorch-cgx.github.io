---
layout: page
title: Usage
permalink: /usage
---

To enable `cgx` in the training the user has to 
import the `torch_cgx` extension and specify `cgx` as `torch.distributed.init_process_group` backend parameter.

Example:
``` python
import torch
import torch.distributed as dist
import torch_cgx

dist.init_process_group('cgx', init_method='env://', rank=args.local_rank)
```

As long as the extension is based on MPI backend, it requires MPI-compliant launcher (`torch.distributed.launch` won't work):
```bash
$ mpirun -np 2 python train.py
```

Also, if your training script was run previously with `torch.distributed.launch` utility, due to MPI launcher you need to set an environment variables (see cifar_train.py in examples)
```
if "OMPI_COMM_WORLD_SIZE" in os.environ:
    os.environ['MASTER_ADDR'] = '127.0.0.1'
    os.environ['MASTER_PORT'] = '4040'
    os.environ["WORLD_SIZE"] = os.environ["OMPI_COMM_WORLD_SIZE"]
    os.environ["RANK"] = os.environ["OMPI_COMM_WORLD_RANK"]
```


# Tuning
CGX by default does not apply compression, it has to be enabled by setting the corresponding environment variable.
CGX is tuned by setting the following environment variables:

- `CGX_COMPRESSION_QUANTIZATION_BITS` - number of bits each value of buffer is quantized to (from 1 to 8). Default is 32 which means no quantization is applied. This variable must be used if the `cgx_hook` communication hook is not registered.
- `CGX_COMPRESSION_BUCKET_SIZE` - size of subarray into which buffer is split before quantization. Default is 512.
- `CGX_COMPRESSION_SKIP_INCOMPLETE_BUCKETS` - boolean variable (0 or 1). After the splitting buffer into buckets, some values of buffer may remain. The variable tells quantization algorithm to compress or not to compress the remaining values. Default 0.
- `CGX_COMPRESSION_MINIMAL_SIZE` - minimal size of buffer (number of elements) to compress. Default is 0 but in fact minimal size is forced to be not less than 16.
- `CGX_FUSION_BUFFER_SIZE_MB`. CGX is leveraging [Tensor Fusion](https://github.com/horovod/horovod#tensor-fusion), a performance feature introduced in Horovod. This feature batches small allreduce operations. This decreases a latency in Data Parallel training. The environment variable controls the size of maximal buffer (in MB) that is communicated within one iteration of allreduce algorithm. Default is 64.
- `CGX_INNER_COMMUNICATOR_TYPE`. Specifies what library to use as communication backend for intra node communication (MPI, SHM, NCCL). Default is SHM.
- `CGX_CROSS_COMMUNICATOR_TYPE`. Specifies what library to use as communication backend for inter node communication (MPI, NCCL). Default is MPI.
- `CGX_INTRA_BROADCAST`. boolean variable (0 or 1). Parameter for multinode training. When enabled, inter-node communication is performed by only one gpu per node.

All the variables, except `CGX_COMPRESSION_QUANTIZATION_BITS` and `CGX_COMPRESSION_BUCKET_SIZE`  must be set **before** loading the module (`import torch_cgx`).

# Layerwise compression

Also, it order to perform layerwise compression and being able to filter small sensitive to gradient compression
layers (typically these are batch norm layers and biases) the `cgx` needs to have information about the model.
For that users need to register the communication hook. The minimal size of the layers can be
controlled with `layer_min_size` parameter.

``` python
model = torch.
from cgx_utils import cgx_hook, CGXState
state = CGXState(torch.distributed.group.WORLD, layer_min_size=1024,
                  compression_params={"bits": quantization_bits,
                                      "bucket_size": quantization_bucket_size})
model.register_comm_hook(state, cgx_hook)
``` 

Where `quantization_bits` and  `quantization_bucket_size` are parameters for max-min quantizer (QSGD).  
These values, if set, overwrite the values of  `CGX_COMPRESSION_QUANTIZATION_BITS` and `CGX_COMPRESSION_BUCKET_SIZE`
environment variables.



