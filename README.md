Perplexity MoE Kernels
==========

# Installation

```bash
cd pplx-kernels
TORCH_CUDA_ARCH_LIST=9.0a+PTX python3 setup.py bdist_wheel
pip install dist/*.whl
```

# Testing

To build the C++ tests and benchmarks:

```bash
cd pplx-kernels
mkdir build-cmake
cd build-cmake

TORCH_PREFIX_PATH=$(python3 -c 'import torch; print(torch.utils.cmake_prefix_path)')

cmake ../csrc \
    -GNinja \
    -DCMAKE_PREFIX_PATH=$TORCH_PREFIX_PATH \
    -DTORCH_CUDA_ARCH_LIST=9.0a+PTX \
    -DWITH_TESTS=ON \
    -DWITH_BENCHMARKS=ON

ninja test_all_to_all bench_all_to_all
```

To run the all-to-all tests on one node:

```bash
NVSHMEM_REMOTE_TRANSPORT=none mpirun -np 4 ./test_all_to_all
```


To run the all-to-all benchmarks on one node:

```bash
NVSHMEM_REMOTE_TRANSPORT=none mpirun -np 4 ./bench_all_to_all
```


# Inter-Node Benchmarks

To test on a 32-device cluster spread across 4 nodes, run the following command on all nodes, alternating the rank from 0 to 3 and setting the master address to point to the rank-0 node:

```bash
export NODE_RANK=<rank>
export WORLD_SIZE=32
export WORLD_LOCAL_SIZE=8
export MASTER_ADDR=<master-address>
export MASTER_PORT=29500
export NVSHMEM_IB_ENABLE_IBGDA=1
python3 -m tests.bench_all_to_all
```

# Benchmark Results

1 token per GPU:

|    1 tok per GPU   |       EP128       |       EP64       |       EP32       |       EP16       |       EP8       |
|:------------------:|:-----------------:|:----------------:|:----------------:|:----------------:|:---------------:|
|   NVLINK Dispatch  | x                 | x                | x                | x                | 41.6μs ±  1.3μs |
|   IBGDA Dispatch   | 125.9μs ±  0.6μs  | 121.0μs ±  0.2μs | 115.7μs ±  1.4μs | 102.7μs ±  8.7μs | x               |
|    IBRC Dispatch   | 488.4μs ± 51.0μs  | 525.0μs ±  9.4μs | 421.2μs ± 35.5μs | 290.5μs ±  4.7μs | x               |
|   NVLINK Combine   | x                 | x                | x                | x                | 41.7μs ±  3.0μs |
|    IBGDA Combine   | 63.2μs ±  8.3μs   | 58.6μs ±  1.0μs  | 55.4μs ±  0.8μs  | 62.7μs ±  0.7μs  | x               |
|    IBRC Combine    | 786.8μs ± 149.8μs | 400.0μs ± 47.9μs | 122.1μs ± 38.2μs | 85.9μs ±  5.3μs  | x               |
|      Torch AtA     | 132.0μs ± 25.9μs  | 101.6μs ± 15.7μs | 95.7μs ± 14.3μs  | 109.7μs ±  3.1μs | 24.4μs ± 16.3μs |
| NVLINK NVSHMEM AtA | x                 | x                | x                | x                | 59.9μs ± 30.7μs |
|  IBGDA NVSHMEM AtA | 132.4μs ± 73.3μs  | 95.3μs ± 23.5μs  | 77.3μs ± 23.0μs  | 71.7μs ± 14.6μs  | x               |
|  IBRC NVSHMEM AtA  | 258.8μs ± 145.3μs | 98.9μs ± 57.1μs  | 63.2μs ± 20.3μs  | 55.4μs ± 12.6μs  | x               |


128 tokens per GPU:

|   128 tok per GPU  |        EP128       |        EP64        |        EP32        |        EP16       |        EP8        |
|:------------------:|:------------------:|:------------------:|:------------------:|:-----------------:|:-----------------:|
|   DeepEP Dispatch  | 192μs              | 186μs              | 182μs              | 173μs             | 163μs             |
|   NVLINK Dispatch  | x                  | x                  | x                  | x                 | 83.6μs ±  1.0μs   |
|   IBGDA Dispatch   | 307.7μs ±  3.0μs   | 317.4μs ±  1.5μs   | 427.6μs ±  1.4μs   | 622.4μs ±  1.7μs  | x                 |
|    IBRC Dispatch   | 2038.5μs ± 77.0μs  | 1669.3μs ± 64.0μs  | 973.5μs ± 37.9μs   | 687.1μs ± 12.9μs  | x                 |
|   DeepEP Combine   | 369μs              | 353μs              | 350μs              | 329μs             | 318μs             |
|   NVLINK Combine   | x                  | x                  | x                  | x                 | 102.3μs ±  0.6μs  |
|    IBGDA Combine   | 593.9μs ±  6.6μs   | 529.9μs ±  6.7μs   | 481.4μs ±  3.6μs   | 668.1μs ±  3.4μs  | x                 |
|    IBRC Combine    | 1184.8μs ± 79.7μs  | 1058.5μs ± 49.6μs  | 916.5μs ± 45.1μs   | 633.4μs ± 14.0μs  | x                 |
|      Torch AtA     | 4972.0μs ± 135.8μs | 5418.1μs ± 241.4μs | 4225.9μs ± 69.5μs  | 3213.9μs ± 19.7μs | 699.9μs ±  2.2μs  |
| NVLINK NVSHMEM AtA | x                  | x                  | x                  | x                 | 6585.3μs ±  2.4μs |
|  IBGDA NVSHMEM AtA | 6180.1μs ± 344.7μs | 6916.3μs ± 315.4μs | 4603.4μs ± 133.1μs | 3444.8μs ± 15.3μs | x                 |
|  IBRC NVSHMEM AtA  | 6378.5μs ± 375.9μs | 6625.1μs ± 371.3μs | 4371.3μs ± 148.8μs | 3410.1μs ± 20.2μs | x                 |
