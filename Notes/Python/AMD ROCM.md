First connect to gatech VPN

In terminal: `ssh amd-gt-nodes.com`

-OR-

Connect directly with vs code ssh (cmd + shift + p) <------- **Recommended way to connect**

# Connected to CPU Head Node

Connect to GPU node (for actual work) with `gpu-shell`
+ If a GPU is available one is automatically allocated for you to use otherwise you will be queued
+ Exit the GPU node with `exit` once you are done to release a GPU for somebody else to use

`gt-help` shows a list of commands (done in CPU head node)

==if server gets stuck run `killall -u chsu323` in CPU head node and reload==

sometimes the server just decides to kill itself for god knows what reason. If repeatedly restarting doesn't fix anything try checking the class piazza to see if anyone is having problems. godspeed.

| For student workloads, the current scheduling limits include:  <br>  <br>  * Maximum GPUs per job:       1  <br>  * Maximum CPU cores per job: 16  <br>  * Maximum system RAM:        236 GB  <br>  * Maximum running jobs:        1  <br>  * Maximum submitted jobs:      5  <br>  * Maximum job duration:        4 hours  <br>  <br>The standard interactive gpu-shell environment requests:  <br>  <br>  * 1 AMD Instinct MI300X GPU  <br>  * 16 CPU cores  <br>  * 96 GB system RAM  <br>  * up to 2 hoursGPU workloads should NOT be run directly on the login/head node.  <br>  <br>To request an interactive GPU session, run:  <br>  <br>    gpu-shell  <br>  <br>The cluster will request a GPU through Slurm.  <br>  <br>If a GPU is immediately available, you will enter the GPU session. If the cluster is busy, your request may remain queued until a GPU becomes available.  <br>  <br>Once you are inside the GPU session, run:  <br>  <br>    gpu-test  <br>  <br>This performs a basic test of your allocated AMD Instinct MI300X GPU environment.  <br>  <br>When finished, leave the GPU session with:  <br>  <br>    exit  <br>  <br>Exiting the GPU session releases the GPU for another student. | USEFUL COMMANDS  <br>===============  <br>  <br>    gt-help  <br>        Show cluster help and available student tools.  <br>  <br>    gt-nodes-status  <br>        Show the current GPU-node status.  <br>  <br>    gpu-shell  <br>        Request an interactive GPU session.  <br>  <br>    gpu-shell --status  <br>        Check the status of your interactive GPU request.  <br>  <br>    gpu-shell --cancel  <br>        Cancel a queued interactive GPU request.  <br>  <br>    gt-running-jobs  <br>        Show your currently running Slurm jobs.  <br>  <br>    gt-queued-jobs  <br>        Show your queued Slurm jobs.  <br>  <br>    gt-last-job  <br>        Show information about your most recent job.  <br>  <br>    gpu-test  <br>        Run a basic GPU environment test from inside a GPU allocation. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

# Programming within GPU-Shell

Use the following to double check that you are ACTUALLY running on the GPU:

```
def part0_device_report() -> dict:

	return {
	
	"torch_version": torch.__version__,
	
	"cuda_available": torch.cuda.is_available(),
	
	"device_name": torch.cuda.get_device_name(0) if torch.cuda.is_available() else "",
	
	"hip_version": getattr(torch.version, "hip", None),
	
	}
```

**PyTorch runs an operation wherever its input tensors live: on the CPU by default, and on the GPU only if the tensors were moved there first.**

A tensor's `.to("cuda")` method returns a copy on the GPU. Under ROCm the device is still called "cuda". `C.device` tells you where a result lives. Print it if you are unsure.
+ Use `.to("cuda")` to move tensors to GPU so computation can be done on GPU

# Built in HIP variables

When writing GPU kernels in AMD's HIP (Heterogeneous-Compute Interface for Portability) using ROCm, the built-in variables define the **execution geometry** of your kernel.

GPU execution is heavily parallel. Instead of writing a loop to process arrays, you write a single function (a kernel) that is executed simultaneously by thousands of independent "threads." Built-in variables are how each individual thread determines _who_ it is and _what data_ it should process.

## 1. The Execution Hierarchy (Grid, Block, Thread)

To understand the built-in variables, you must understand how HIP groups work:

- **Grid:** The entire launch space of your kernel.
+  **Block:** A subgroup of threads within the Grid. Threads in the same block can share fast local memory (`__shared__`) and synchronize with each other.
- **Thread:** The smallest unit of execution. Every thread runs the exact same code but uses built-in coordinates to pick a unique piece of data.
## 2. Coordinate Built-in Variables

HIP provides four main built-in variables to determine coordinates. These are structs of type `dim3`, meaning they have three components: `.x`, `.y`, and `.z`

| **Variable** | **Description** | **Meaning**                                                |
| ------------ | --------------- | ---------------------------------------------------------- |
| `threadIdx`  | Thread Index    | The coordinate of the current thread **within its block**. |
| `blockIdx`   | Block Index     | The coordinate of the current block **within the grid**.   |
| `blockDim`   | Block Dimension | The total size (number of threads) of a block.             |
| `gridDim`    | Grid Dimension  | The total size (number of blocks) of the grid.             |

Because these are 3D structures, you access them depending on how you launched your kernel:
- **1D launch:** Use `.x` (e.g., `threadIdx.x`)
- **2D launch:** Use `.x` and `.y` (useful for images or matrices)
- **3D launch:** Use `.x`, `.y`, and `.z` (useful for volumetric data)
## 3. Hardware Architecture Variables

| **Variable** | **Type** | **Description**                                                                                                   |
| ------------ | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `warpSize`   | `int`    | The number of threads that execute instructions synchronously in hardware (called a "Wavefront" on AMD hardware). |

**AMD Note:** On NVIDIA hardware, `warpSize` is strictly 32. On AMD hardware, `warpSize` returns **64** on CDNA architectures (Instinct accelerators like MI100, MI200, MI300) and **32** on modern RDNA architectures (Radeon gaming GPUs like RX 6000 and 7000 series).
## 4. Built-in Function Execution Qualifiers

You must prepend these keywords to tell the compiler where a function will run and from where it can be called.
- `__global__`: The function is a **Kernel**. It is launched from the CPU (Host) but executes on the GPU (Device). _Must return `void`._
- `__device__`: A helper function that executes on the GPU and can _only_ be called by other GPU functions.
- `__host__`: A normal CPU function (this is the default if no qualifier is present).
- `__host__ __device__`: The compiler will generate two versions of this function—one for the CPU and one for the GPU.
## 5. Memory Space Qualifiers

These tell the GPU where to physically store variables.
- `__shared__`: Memory shared among all threads in a single Block. It is incredibly fast but limited in size (typically 64KB per block).
- `__constant__`: Memory that the CPU writes to before execution, and the GPU reads extremely fast during execution.
- _(default)_: Variables declared inside a kernel without qualifiers are stored in extremely fast registers per thread. Pointers passed into the kernel (like arrays) point to global VRAM.
## 6. Synchronization Built-ins

Because threads run concurrently, sometimes you need them to wait for each other—especially when using `__shared__` memory.
- `__syncthreads()`: Places a barrier. No thread in the **Block** can pass this line of code until all other threads in the block have reached it.
## 7. Syntax & Pattern Examples

### Pattern 1: Calculating a 1D Global Index (The most common pattern)

To find a thread's unique global ID across the entire grid, you multiply its block ID by the block size, and add its thread ID.

C++

```
__global__ void addVectors(float* A, float* B, float* C, int N) {
    // 1. Calculate the unique global index for this thread
    int idx = (blockIdx.x * blockDim.x) + threadIdx.x;
    
    // 2. Perform a bounds check (grids are often padded)
    if (idx < N) {
        // 3. Do the work!
        C[idx] = A[idx] + B[idx];
    }
}
```

### Pattern 2: Calculating a 2D Global Index (For Images/Matrices)

If you are processing a 1920x1080 image, you calculate row and column indices.

C++

```
__global__ void imageProcessor(float* imageArray, int width, int height) {
    // Calculate Column (X axis)
    int col = (blockIdx.x * blockDim.x) + threadIdx.x;
    // Calculate Row (Y axis)
    int row = (blockIdx.y * blockDim.y) + threadIdx.y;
    
    if (col < width && row < height) {
        // Flatten the 2D coordinate back into a 1D array index
        int flat_idx = (row * width) + col;
        
        // Process the pixel
        imageArray[flat_idx] *= 2.0f;
    }
}
```

### Pattern 3: Using Shared Memory & Synchronization

Here is how you use `__shared__` and `__syncthreads()` to cooperatively load data.

C++

```
__global__ void blockSum(float* input, float* output) {
    // Allocate fast shared memory for the block
    __shared__ float localData[256]; 
    
    int globalIdx = (blockIdx.x * blockDim.x) + threadIdx.x;
    int localIdx  = threadIdx.x;
    
    // 1. Every thread collaboratively loads one element from global VRAM to Shared RAM
    localData[localIdx] = input[globalIdx];
    
    // 2. WAIT for all threads in the block to finish loading
    __syncthreads();
    
    // 3. Now it is safe to read data loaded by *other* threads
    if (localIdx == 0) {
        float sum = 0;
        // The first thread sums up the entire block's data
        for(int i = 0; i < blockDim.x; i++) {
            sum += localData[i]; 
        }
        output[blockIdx.x] = sum; // Write block result to output
    }
}
```

## 8. How to Launch a Kernel

To use these functions, you configure the Grid and Block sizes using the `dim3` struct, then launch the kernel using the triple-chevron syntax `<<<grid, block>>>`.

C++

```
#include <hip/hip_runtime.h>

int main() {
    int total_elements = 10000;
    
    // 1. Define block size (often 128, 256, or 512 threads)
    int threadsPerBlock = 256;
    
    // 2. Calculate how many blocks are needed to cover the total elements.
    // Adding (threadsPerBlock - 1) ensures we round up if it's not a perfect multiple.
    int blocksPerGrid = (total_elements + threadsPerBlock - 1) / threadsPerBlock;
    
    // Declare the device pointers
    float *d_A, *d_B, *d_C;
    // ... hipMalloc & hipMemcpy setup omitted for brevity ...
    
    // 3. Launch the kernel: <<<gridDim, blockDim>>>
    addVectors<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, total_elements);
    
    // Wait for the GPU to finish
    hipDeviceSynchronize();
    
    return 0;
}
```