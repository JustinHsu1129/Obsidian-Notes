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


