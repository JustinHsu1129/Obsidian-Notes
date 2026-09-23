# Inside the CU

Key ISA difference is scalar + vector
+ In AMD GPUs scalar work gets its own processor and reg file
	+ Scalar unit job: control flow and addr generation
	+ 4 scalar inst maps to only 4 registers instead of 4 * 32 registers in NVIDIA
	+ SGPRs can be also used for handling branching and data dependencies
		+ Divergence is not branching-it is masking
+ Compiler works with the microarchitecture
	+ Schedules independent work between dependent inst

CU (CDNA4) and WGP (CDNA5) are equivalent things, just in the WGP the SIMDs are 32 and one wavefront inst issued in one cycle
+ Getting improvements to matmul performance by increasing the size each inst can handle between generations

# ROCm Software Stack

AMD's open-source software stack for GPU computing and AI. It serves as the AMD alternative to NVIDIA's CUDA ecosystem.

### High-Level Architecture (Top to Bottom)

1. **Frameworks & User Applications**
   - **AI / Deep Learning**: PyTorch, TensorFlow, JAX, vLLM, Hugging Face, OpenAI Triton.
   - Most modern frameworks support ROCm natively or via prebuilt ROCm wheels/Docker containers.

2. **Domain-Specific Acceleration Libraries**
   - **Dense Linear Algebra (GEMM / Matmul)**:
     - `rocBLAS`: Core Basic Linear Algebra Subprograms (BLAS-3 covers GEMM).
     - `hipBLAS` / `hipBLASLt`: Lightweight, highly tuned GEMM library with FP8/FP16/BF16 support (ideal for LLM inference/training).
     - `Composable Kernel (CK)`: High-performance C++ templated library for GEMM and attention kernels, targeting CDNA matrix cores.
   - **Deep Learning Primitives**:
     - `MIOpen`: AMD's deep learning primitives library (convolutions, pooling, batch norm, activation, attention).
   - **Multi-GPU / Distributed Scaling**:
     - `RCCL` (Radeon Collective Communication Library): Inter-GPU communication (AllReduce, AllGather, Broadcast).
   - **Math & Utilities**:
     - `rocFFT` (Fast Fourier Transforms), `rocSOLVER` (Lapack-like solvers), `rocRAND` (random number generation).

3. **Programming Model: HIP (Heterogeneous-Compute Interface for Portability)**
   - **C++ dialect & API**: Syntax is almost 1:1 identical to CUDA (`hipMalloc`, `hipMemcpy`, kernel launches).
   - **Cross-platform**: HIP code can be compiled to run on **both** AMD GPUs (via ROCm/LLVM) and NVIDIA GPUs (via NVCC).
   - **HIPIFY tools**: Automated source-to-source translation tools (`hipify-clang`, `hipify-perl`) that convert existing CUDA source code into portable HIP C++ code with minimal manual changes.

4. **Compilers & Developer Tools**
   - **Compiler (`hipcc` / `amdclang++`)**: Built on open-source LLVM/Clang backend with AMDGPU code generator target.
   - **Profiling & Debugging**:
     - `rocprof` / `ROCm Systems Profiler (RSP)`: Trace API calls and GPU kernel execution timelines.
     - `Omnitrace` / `Omniperf`: High-level system profiler and kernel-level performance analyzer.
     - `rocgdb`: Source-level GPU debugger.

5. **Low-Level Runtimes & Kernel Drivers**
   - **ROCr Runtime**: Core user-space runtime; manages HSA (Heterogeneous System Architecture) signal queues, kernel dispatching, and virtual memory.
   - **AMDGPU / KFD (Kernel Fusion Driver)**: Linux kernel module handling low-level PCIe communication, memory allocation, page tables, and hardware interrupts.

---

### Quick Cheat Sheet: ROCm vs. CUDA Equivalents

| Layer / Purpose | NVIDIA CUDA | AMD ROCm |
| :--- | :--- | :--- |
| **Programming Language / API** | CUDA C++ | HIP C++ |
| **Compiler** | `nvcc` | `hipcc` / `amdclang++` |
| **GEMM / BLAS** | `cuBLAS` / `cuBLASLt` | `rocBLAS` / `hipBLASLt` |
| **Deep Learning Primitives** | `cuDNN` | `MIOpen` |
| **Distributed / Collectives** | `NCCL` | `RCCL` |
| **Matrix Core Programming** | WMMA / MMA PTX | `rocWMMA` / MFMA instructions |
| **Profiling & Diagnostics** | Nsight Systems / Compute | `rocprof` / `Omniperf` / `Omnitrace` |
| **GPU Management CLI** | `nvidia-smi` | `rocm-smi` / `amd-smi` |
| **Kernel Driver** | `nvidia.ko` | `amdgpu.ko` / `kfd` |

---

### How GEMM Flows Through the Stack

1. **User Request**: High-level code (e.g. PyTorch `torch.matmul(A, B)` or Triton GEMM kernel).
2. **Library Dispatch**: PyTorch calls `hipBLASLt` or `rocBLAS` (or compiles a kernel via Triton/CK).
3. **Instruction Generation**: The compiler emits AMD CDNA machine instructions, specifically **MFMA** (Matrix Fused Multiply-Add) instructions.
4. **Execution on Hardware**: Dispatched through ROCr runtime to the CU/WGP, utilizing the Matrix Cores and local LDS (Local Data Share) scratchpad memory.

## Origami + Stream-K: Choosing a GEMM Kernel Without a Table

In high-performance GPU computing, running a GEMM ($C = A \times B$) is not just about doing math—it is about **picking the best kernel configuration** (tile dimensions, memory layout, workgroup sizes, and unroll factors) for a specific matrix shape $(M, N, K)$.

Historically, this required massive pre-computed **lookup tables**. AMD's **Origami + Stream-K** replaces these tables with **analytical modeling** and **even workload streaming**.

---

### 1. The Old Problem: Why "Tuning Tables" Hurt Modern AI

#### The Classic Approach (Lookup Tables / Autotuning)
Historically, libraries like `rocBLAS` or `cuBLAS` relied on **offline autotuning**:
- Engineers ran benchmarks across tens of thousands of matrix sizes ($M, N, K$) ahead of time.
- The fastest kernel configuration for each shape was saved into a gigantic static lookup table packaged into the library.

#### Why This Breaks Down in Modern AI
1. **Dynamic Shapes in LLMs**: LLMs (in frameworks like vLLM or SGLang) process dynamic batch sizes and variable prompt/decode lengths ($M=1, 7, 33, 128, \dots$). Pre-tuning every conceivable shape is impossible.
2. **Binary Bloat**: Storing tables with millions of kernel variations inflates library and container sizes by gigabytes.
3. **The "Unseen Shape" Penalty**: If an application hits a matrix shape not in the table, the library falls back to a generic, unoptimized heuristic that leaves substantial performance on the table.
4. **Hardware Refresh Burden**: Every new GPU architecture or revision requires weeks of re-running exhaustive tuning sweeps.

---

### 2. Stream-K: Solving the "Wave Quantization" Problem

To understand Origami, we first need to look at **how GEMM work is partitioned on the GPU**.

#### The Problem: Wave Quantization (Tail Effect)
In classic **Data-Parallel GEMM**, the output matrix $C$ is cut into 2D grid tiles (e.g., $128 \times 128$). Each Compute Unit (CU) computes one complete output tile from start to finish.

* **What happens with uneven tile counts?**
  Suppose an AMD MI300X has **304 CUs**, but a matrix shape only produces **305 tiles**:
  - **Round 1 (Wave 1):** All 304 CUs run 304 tiles concurrently (100% utilization).
  - **Round 2 (Wave 2 / Tail):** Exactly **1 CU** calculates the 305th tile, while the other **303 CUs sit completely idle**!
  - **Result:** The whole kernel takes nearly double the time it should have taken, dropping GPU efficiency by ~50%.

```
Classic Data-Parallel (Tail Effect):
Wave 1: [CU 0] [CU 1] [CU 2] ... [CU 303]  <- 100% Full
Wave 2: [CU 0] [ IDLE ] [ IDLE ] ... [ IDLE ]   <- 99.7% Wasted Capacity!
```

#### The Stream-K Solution: Work-Centric Streaming
Instead of assigning whole output tiles to CUs, **Stream-K** treats the entire computation ($M \times N \times K$) as a continuous 1D stream of inner-loop Multiply-Accumulate (MAC) iterations:
1. It divides the total work **equally across all available CUs**, like slicing an exact length of ribbon among workers.
2. Every CU does roughly the **exact same number of iterations**, finishing at virtually the same microsecond.
3. Most tiles are computed entirely within one CU. The few "boundary" tiles that cross between two CUs have their partial sums combined via a fast reduction.
4. **Benefit**: Eliminates the tail effect entirely. Performance scales smoothly and linearly across *any* matrix size without severe cliff drops.

---

### 3. Origami: An Analytical Model Instead of a Table

While Stream-K fixes *work distribution*, the system still needs to choose parameters like tile size ($M_{tile}, N_{tile}$), wave layout, and memory staging depths.

This is where **Origami** enters.

#### What is Origami?
**Origami** is AMD's analytical, deterministic performance model embedded into the GEMM runtime (such as `Tensile` and `hipBLASLt`). 

Instead of looking up historical measurements in a table, Origami **calculates the expected latency** of candidate kernel configurations in real time using physics and microarchitecture formulas:

$$\text{Predicted Latency} = \max(\text{Compute Latency},\, \text{Memory Latency}) + \text{Overhead}$$

#### How Origami Predicts Kernel Performance
In microseconds, Origami evaluates candidate configurations against the hardware’s microarchitectural specs:
- **Compute Latency**: How many cycles will the MFMA (Matrix Core) instructions take for this tile size?
- **Memory Latency**:
  - How much data must be read from High Bandwidth Memory (HBM)?
  - What is the expected L2 cache hit rate for this traversal pattern?
  - Will LDS (Local Data Share) bank conflicts occur?
- **Occupancy & Registers**: Can the CU fit the requested vector registers (VGPRs) and scalar registers (SGPRs) without register spilling?

Origami scores the candidates and deterministically picks the winner **at runtime with virtually zero overhead**.

---

### 4. Why Origami + Stream-K Are the Perfect Match

| Component | Role | What It Solves |
| :--- | :--- | :--- |
| **Stream-K** | **Execution Engine** | Distributes GEMM iterations evenly across CUs; eliminates wave quantization / idle GPU tails. |
| **Origami** | **Brain / Planner** | Mathematically estimates the fastest tile sizes and parameters on-the-fly; eliminates static tuning tables. |

#### Why They Work So Well Together:
Traditional tile-based GEMMs have erratic, step-function performance curves because of wave quantization, making them hard for mathematical formulas to model without empirical measurements.

Because **Stream-K makes performance smooth, continuous, and predictable**, Origami's analytical formulas can model the execution time with extreme accuracy.

---

### 5. Practical Impact & Usage in ROCm

- **Zero-Day Hardware Support**: New GPU stepping or custom clock rate? No need to re-run 48-hour tuning sweeps; Origami recalculates the optimal configuration immediately based on hardware parameters.
- **Drastically Smaller Footprint**: Eliminates tens of megabytes of pre-compiled GEMM selection tables from `hipBLASLt` binaries.
- **First-Class in Modern Instinct GPUs**:
  - In `hipBLASLt` on **MI300 / MI350**, Origami + Stream-K is the primary strategy for achieving out-of-the-box peak TFLOPS.
  - Can be observed or tuned via ROCm environment variables:
    ```bash
    # Set Tensile selection method (e.g., enable Origami with Stream-K)
    export TENSILE_SOLUTION_SELECTION_METHOD=2
    ```
- **Crucial for LLM Serving**: Enables inference engines (like vLLM on AMD) to maintain sustained, high compute efficiency even as user prompt lengths and token counts fluctuate wildly on every forward pass.

## Mapping GEMM onto Hardware (AMD CDNA Deep Dive)

How does a mathematical matrix formula ($C = A \times B + C$) actually turn into physical electrons moving through an AMD Instinct GPU (such as the MI250, MI300X, or MI350)?

To achieve near-peak theoretical TFLOPS, a GEMM is decomposed hierarchically so that data flows naturally through both the **hardware execution hierarchy** (from the whole chip down to individual matrix ALUs) and the **memory hierarchy** (from HBM down to registers).

---

### 1. The Hierarchical Mapping: From Matrix to Silicon

At every level of the GEMM algorithm, a piece of the matrix is paired with a specific AMD hardware structure:

```
[ Entire Matrix C ]             ───►  Whole GPU (e.g. MI300X with 8 XCDs / 304 CUs)
        │
        ▼  (Split into Macro-Tiles)
[ Workgroup Tile (e.g. 256x256) ] ──►  Single Compute Unit (CU) / WGP (coordinated via LDS)
        │
        ▼  (Split into Sub-Tiles)
[ Wavefront Tile (e.g. 64x64) ]   ──►  Single Wavefront (64 threads executing in lockstep)
        │
        ▼  (Split into Micro-Tiles)
[ MFMA Instruction (e.g. 32x32) ] ──►  CDNA Matrix Core Unit (hardware systolic matrix instruction)
```

| Logical GEMM Concept | AMD Hardware Unit | What It Does |
| :--- | :--- | :--- |
| **Full Matrix Grid** | **GPU / Accelerator Complex (XCDs)** | The entire grid of $M \times N$ tiles is distributed across all available CUs on the GPU chiplets. |
| **Workgroup Tile (Macro-Tile)** | **Compute Unit (CU) / WGP** | A team of cooperating threads (typically 256–512 threads, or 4–8 wavefronts) calculates one output block of $C$ (e.g., $256 \times 256$). |
| **Shared Scratchpad Cache** | **Local Data Share (LDS)** | On-chip high-speed SRAM (64 KB per CU). Caches slices of inputs $A$ and $B$ along the $K$-dimension so all wavefronts in the CU can reuse them without re-reading HBM. |
| **Wavefront Tile (Sub-Tile)** | **Wavefront (64 threads)** | One AMD wavefront (Wave64) computes a portion of the workgroup tile (e.g., $64 \times 64$). |
| **Accumulator Storage** | **AccVGPRs (Accumulator Registers)** | Dedicated register file on CDNA GPUs that stores the running FP32 sum across the $K$-loop without eating up standard vector registers. |
| **Micro-Tile (Matrix Multiply)** | **Matrix Core Unit (MFMA Engine)** | Dedicated hardware matrix multiplier that executes hardware instructions (like `v_mfma_...`) directly in silicon. |

---

### 2. The Dataflow Path: From Memory to Matrix Cores

To sustain high compute efficiency, the memory subsystem stages data in closer, faster memories at each step:

```
[ High Bandwidth Memory (HBM3/HBM3e) ]  ~5.3 TB/s (Holds full matrices A, B, and C)
                   │
                   ▼  (Stream-K / Cache-aware tiling)
[ L2 Cache Shared Fabric ]              High hit rates, absorbs repeated matrix tile reads
                   │
                   ▼  (Global Vector Loads: buffer_load / global_load)
[ Local Data Share (LDS) ]              64 KB per CU SRAM; acts as a local staging tile
                   │
                   ▼  (Local LDS Reads: ds_read)
[ Vector Registers (VGPR) ]             Holds input fragments (A_frag, B_frag) for the wavefront
                   │
                   ▼  (Hardware MFMA instruction)
[ CDNA Matrix Cores (ALU) ]             Hardware multiplies matrix slices in a few clock cycles
                   │
                   ▼  (Direct accumulation)
[ Accumulator Registers (AccVGPR) ]     Holds running sum C in silicon across all K-iterations
```

---

### 3. Inside the Wavefront: How MFMA Instructions Work

On NVIDIA GPUs, Tensor Cores operate on **Warp32** (32 threads). On AMD CDNA GPUs, compute wavefronts are **Wave64** (64 threads).

#### What is an MFMA Instruction?
**MFMA** stands for **Matrix Fused Multiply-Add**. It is an assembly-level instruction (such as `v_mfma_f32_32x32x8_fp16`) that commands the CU's dedicated Matrix Core to execute:

$$\mathbf{D} = \mathbf{A} \times \mathbf{B} + \mathbf{C}$$

* **How 64 threads collaborate:**
  - An individual thread does **not** multiply a matrix alone.
  - Instead, all 64 threads in the wavefront pass their private register values (**VGPRs**) into the Matrix Core simultaneously.
  - The Matrix Core wires the 64 lanes together, performs the dot products across its internal systolic multiplier array, and deposits the results into the destination registers.

#### Common MFMA Instructions:
- `v_mfma_f32_32x32x8_fp16`: Multiplies a $32 \times 8$ tile of $A$ (FP16) by an $8 \times 32$ tile of $B$ (FP16) and accumulates the result into a $32 \times 32$ tile of **FP32** output across the 64 lanes.
- `v_mfma_f32_16x16x32_f8`: Multiplies a $16 \times 32$ tile of FP8 by a $32 \times 16$ tile of FP8 into FP32 accumulators—the bedrock instruction for fast LLM inference on MI300X and MI350.

---

### 4. The Inner Loop: Latency Hiding via Ping-Pong Buffering

Matrix multiplication is fundamentally an inner-loop reduction along the $K$-dimension:
$$C_{i, j} = \sum_{k=0}^{K-1} A_{i, k} \cdot B_{k, j}$$

Because $K$ can be thousands of elements long (e.g. hidden dimension $K = 8192$ in LLMs), the CU cannot load all of $A$ and $B$ into LDS at once. It processes $K$ in small slices called $K_{\text{step}}$ (e.g., 32 or 64 elements at a time).

To prevent the Matrix Cores from stalling while waiting for the next slice from memory, AMD GEMM kernels use **Double Buffering (Ping-Pong Buffering)**:

```
Cycle Timeline:
─────────────────────────────────────────────────────────────────────────────
Step N:   [ Matrix Core: Runs MFMA on Buffer 0 in VGPRs ]
          [ Memory Units: Concurrently fetches next K-slice into LDS Buffer 1 ]
          [ Scalar ALU: Calculates next memory addresses and loop branches ]
─────────────────────────────────────────────────────────────────────────────
Step N+1: [ Matrix Core: Runs MFMA on Buffer 1 in VGPRs ]
          [ Memory Units: Concurrently fetches next K-slice into LDS Buffer 0 ]
          [ Scalar ALU: Calculates next memory addresses and loop branches ]
─────────────────────────────────────────────────────────────────────────────
```
Because the Matrix Cores compute on one buffer while memory units fetch into the other, **compute latency hides memory latency completely**.

---

### 5. AMD-Specific Architectural Advantages in GEMM

1. **Dedicated Accumulator VGPRs (AccVGPRs)**
   - In CDNA, AMD physically divided vector registers into regular **VGPRs** (for loading and holding inputs) and **AccVGPRs** (for holding accumulator results).
   - *Why this matters*: The accumulator tile ($C$) must remain alive in registers through the entire loop. By giving accumulators their own register bank, the kernel can hold massive $128 \times 128$ accumulator tiles per wavefront without running out of standard VGPRs or suffering from **register spilling** to scratch memory.

2. **Offloading Bookkeeping to the Scalar Unit (SALU + SGPR)**
   - As highlighted in *Inside the CU*, AMD GPUs have a separate scalar processor per CU.
   - While the Matrix Cores crunch through intensive `v_mfma` instructions, the **Scalar Unit** operates in parallel:
     - Updates global and LDS memory pointers.
     - Handles loop counters, offsets, and branching.
     - *Result*: Zero matrix-core cycles are wasted on loop overhead or pointer arithmetic.

3. **Multi-Die / XCD Affinity (e.g., MI300X)**
   - The MI300X accelerator contains **8 XCDs (Accelerator Complex Dies)** connected by high-speed Infinity Fabric.
   - Modern ROCm GEMM libraries (`hipBLASLt` and Composable Kernel) use **NUMA-aware / die-aware workgroup scheduling**:
     - Workgroups are mapped so that CUs on XCD 0 primarily access the L2 cache partitions and HBM memory channels physically wired to XCD 0.
     - This keeps memory traffic local, avoiding inter-die fabric congestion and maximizing effective memory bandwidth.

