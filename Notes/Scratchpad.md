| Configuration (`per_thread`, `threads_per_block`) | Mean Runtime (ms) | Median Runtime (ms) | Workgroup Size | Arch VGPR | SGPR | Max Waves / CU | Wavefront Occupancy |
| :------------------------------------------------ | :---------------- | :------------------ | :------------- | :-------- | :--- | :------------- | :------------------ |
| `(64, 256)`                                       | 4.613 ms          | 4.553 ms            | 256            | 32        | 32   | 32             | 100.0%              |
| `(64, 32)`                                        | 5.001 ms          | 4.983 ms            | 32             | 32        | 32   | 16             | 50.0%               |
| `(64, 1024)`                                      | 4.701 ms          | 4.734 ms            | 1024           | 32        | 32   | 32             | 100.0%              |
| `(16, 256)`                                       | 4.811 ms          | 4.824 ms            | 256            | 32        | 32   | 32             | 100.0%              |

__global__ void multiply_add_coalesced(const float *__restrict__ A, const float *__restrict__ B, const float *__restrict__ C, float *__restrict__ dst, const int len)

{

// Part 1.6 optimization - each block is doing only 1 calculation per iteration, so what if it did 4 calculations per loop, that way less load/store instructions need to be sent

  

int stride = blockDim.x * gridDim.x;

int start = (blockIdx.x * blockDim.x) + threadIdx.x;

  

for (int idx = start; idx < len; idx = idx + stride)

{

for (int jdx = start; jdx < 4; jdx++)

{

dst[idx[jdx]] = A[idx[jdx]] * B[idx[jdx]] + C[idx[jdx]];

}

}

}

void launch_multiply_add_coalesced(const float *A, const float *B, const float *C,

float *dst,

const int len,

const int blocks,

const int threads_per_block)

{

multiply_add_coalesced<<<blocks, threads_per_block>>>(A, B, C, dst, len);


