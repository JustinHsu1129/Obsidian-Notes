# AMD ROCm / HSA Execution & Hardware Architecture

## 1. Programming Model vs. Silicon Reality (SIMT vs. SIMD)
* **SIMT (Single Instruction, Multiple Threads):** The software programming abstraction exposed to developers (HIP/CUDA). Code is written from the perspective of an individual scalar thread (`tid`) operating on single variables.
* **SIMD (Single Instruction, Multiple Data):** The physical execution engine in silicon. Independent hardware cores do not exist per thread; instead, wide pipelined vector execution units execute operations across multiple data lanes simultaneously.
* **The Bridge:**
  * **Thread:** Maps directly to a single **SIMD lane** within an execution unit.
  * **Warp / Wavefront:** A collection of threads (32 threads on NVIDIA warps; 32 or 64 threads on AMD wavefronts) locked together to share a single Program Counter (PC) and instruction decoder.
  * **Branch Divergence:** Handled via hardware execution masks (`EXEC`). When paths diverge (`if/else`), non-qualifying lanes are disabled via mask bits while the vector ALU executes each branch serially.

---

## 2. Kernel Execution Hierarchy

$$\text{Grid} \longrightarrow \text{Thread Blocks (Workgroups)} \longrightarrow \text{Wavefronts (Warps)} \longrightarrow \text{Threads (SIMD Lanes)}$$

| Hierarchy Level | Software Construct | Hardware Target | Memory & Synchronization Scope |
| :--- | :--- | :--- | :--- |
| **Grid** | Complete kernel launch | Entire GPU | Global DRAM, L2 / Infinity Cache access across all CUs. |
| **Thread Block** | Group of wavefronts | Compute Unit (CU) | Shared scratchpad (LDS / Shared Memory) and barrier synchronization (`__syncthreads()`). |
| **Wavefront** | Lockstep group of 32/64 threads | SIMD Vector Engine | Executed as an atomic unit by the CU instruction scheduler. |
| **Thread** | Single scalar execution context | Physical SIMD Lane | Private slice of the Vector Register File (VGPRs). |

### Built-in Thread Indexing (1D Grid)
* `gridDim.x`: Total number of thread blocks in the grid.
* `blockIdx.x`: Index of the current thread block within the grid.
* `blockDim.x`: Number of threads per block.
* `threadIdx.x`: Local thread index within its block.

$$\text{Global Thread ID (tid)} = (\text{blockIdx.x} \times \text{blockDim.x}) + \text{threadIdx.x}$$

---

## 3. Compute Unit (CU) Microarchitecture & Scheduling
* **SIMD Pipelines:** Each CU contains multi-stage pipelined ALUs that deliver single-cycle throughput for vector arithmetic operations once full.
* **Latency Hiding:** Memory operations to VRAM take hundreds of cycles. The CU maintains multiple active wavefronts in flight and switches contexts with zero overhead when a wavefront stalls.
* **Greedy-Then-Oldest (GTO) Scheduling:**
  * **Greedy:** The warp scheduler issues consecutive instructions from the *same* wavefront as long as its dependencies are met to maximize register and cache locality.
  * **Then-Oldest:** When the active wavefront stalls (e.g., waiting on memory loads), the scheduler dispatches instructions from the *oldest* ready wavefront that has waited longest.

---

## 4. Control Plane: Queues, Packets, & Doorbells (HSA Architecture)

AMD ROCm avoids OS kernel transitions (`ioctl` overhead) during dispatch by providing direct user-space submission to the GPU command processor.

- **AQL Packet (Architected Queuing Language):** A fixed 64-byte command packet written into memory. Contains kernel object pointers, grid/block dimensions, completion signal addresses, and kernel argument references.
    
- **HSA Queue:** A ring buffer allocated in shared, cache-coherent memory mapped directly to user space. Multiple user applications can own dedicated queues without interacting with the OS kernel driver.
    
- **Doorbell:** A Memory-Mapped I/O (MMIO) hardware register located on the GPU. Writing the updated queue write-pointer index to this register signals the GPU that new work is pending without polling overhead.
    
- **Command Processor (CP):** On-chip micro-controller that consumes AQL packets from queues, sets up dispatch state, and schedules thread blocks across available CUs.
    
- **Completion Signals:** Atomic memory primitives updated by the GPU hardware when kernel execution completes, allowing the CPU to synchronize without traditional hardware interrupts.