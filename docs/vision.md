# The Tensorplane & `m-store` Manifesto

## Executive Summary: The Monadic Taxonomy
The era of monolithic, kernel-bound operating systems is over. To meet the exascale demands of autonomous AI, we introduce a disaggregated, hardware-accelerated computing continuum based on the **Monadic Taxonomy**. 

## 1. The Brain & Body
*   **Tensorplane (The Agentic Control Plane / "The Brain"):** A Recursive Self-Improving AI Foundry driven by a Multi-Agent System (MAS). Utilizing the **Model Context Protocol (MCP)**, it autonomously monitors **eBPF-driven telemetry** (via tools like Cilium and Tetragon), performs dynamic GPU fractioning, and hot-swaps newly generated C++/Rust kernels into the data plane.
*   ⚡ ⚛️ **`m-store` (The Unified Memory & State Orchestration Fabric / "The Body"):** A zero-kernel, RDMA-first data plane that eliminates the AI "memory wall" and the I/O bottlenecks of the Linux Virtual File System (VFS). It bridges physical GPU VRAM, local NVMe, and **CXL 3.1 Global Integrated Memory** into a single, lock-free **Silicon Data Substrate**.

## 2. The AI Workload Core
Legacy storage forces applications to interact with files. `m-store` discards this abstraction, acting as a highly specialized Memory & State Orchestration Fabric built to feed compute-starved GPUs directly. 

*   **The Inference Substrate (vLLM & SGLang):** `m-store` natively supports complex KV cache abstractions. It acts as the shared KV-cache backend for **vLLM's PagedAttention**, routing memory vectors directly into persistent **Triton attention kernels** via PCIe, completely bypassing the CPU bounce buffer. For multi-step reasoning, it intercepts **SGLang's RadixAttention** LRU evictions, persisting prefix trees to high-speed NVMe to eliminate "memory amnesia" during complex agentic loops. It also streams inactive Mixture of Experts (MoE) parameters directly to GPU VRAM utilizing **GPUDirect Storage (GDS)**.
*   **The Training Substrate (Dataloaders & Checkpointing):** Modeled after architectures like DeepSeek's 3FS, `m-store` separates metadata (managed via FoundationDB) from bulk data chunks (replicated via **Chain Replication with Apportioned Queries [CRAQ]**). It routes massive sequential writes for distributed checkpointing into lock-free hardware queues, minimizing GPU idle time.
*   **The Agentic Memory & Vector Core (MCP & RAG):** As autonomous agents execute complex, multi-stage workflows, their active context windows quickly bloat. `m-store` provides **Agentic Episodic Memory (Engrams)** natively integrated with the Model Context Protocol (MCP), allowing agents to offload and retrieve state instantly. When traversing massive vector graphs (like DiskANN), `m-store` allows continuous ~8KB pointer-chasing reads to bypass the kernel entirely, achieving sub-millisecond retrieval latencies without VFS page-cache thrashing.

## 3. The Agentic Memory Spectrum
As AI evolves into autonomous agents, `m-store` serves as the foundational persistence layer, mapping the entire spectrum of agentic memory directly to silicon:
*   **Short-Term (Working) Memory:** Ephemeral session context is structured as KV Cache Tensors, flushed from GPU VRAM to lock-free NVMe queues instantly when an agent pauses.
*   **Long-Term (Semantic) Memory:** General enterprise knowledge is stored as high-dimensional embeddings. `m-store` navigates these massive graph indexes at bare-metal speeds.
*   **Episodic Memory (Engrams):** Agent actions and memories are recorded as immutable event logs (similar to Amazon Aurora's log-structured state), allowing agents to perform semantic searches over an append-only log.

## 4. The Zero-Copy Data Plane & Transparent POSIX
To achieve microsecond latency, `m-store` utilizes a 100% lock-free, thread-per-core execution model that bridges user-space block devices (BDUS) and transparent zero-copy I/O (zIO).
*   **`Ior` & `Iov` Queues:** Shared ring buffers and registered memory regions that create a true zero-copy data path, allowing data to move directly from the network or disk into the application's memory space without CPU intervention.
*   **Transparent POSIX Interception:** Legacy AI applications do not need to be rewritten. By utilizing `LD_PRELOAD` trampolines and architectures inspired by Direct-FUSE and zIO, `m-store` intercepts standard filesystem calls and routes them directly into the zero-copy NVMe queues.

## 5. Cloud-Native Tiering and Statelessness
While the hot tier relies on bare-metal NVMe, `m-store` natively integrates with AWS S3 and Azure Blobstore to provide multi-cloud durability.
*   **Thread Isolation Architecture:** Standard cloud SDKs rely on TCP/IP and HTTP, which introduce massive context switches. `m-store` isolates cloud I/O to a dedicated background thread pool running an asynchronous runtime. 
*   **DPU Offloading:** Where hardware permits, the TCP/HTTP object-storage client is offloaded to a Data Processing Unit (DPU) like the NVIDIA BlueField-3. The DPU manages the heavy network stack, presenting cloud storage to the host CPU as a local NVMe block device.

## 6. Hardware-Software Co-Design: RISC-V & The Compiler Continuum
`m-store` pushes computation down to the data and hardwires OS services directly into silicon (akin to the ChamelIoT framework). It achieves deterministic, lock-free performance by aggressively leveraging customized RISC-V instruction set extensions:
*   **RISC-V Weak Memory Ordering (RVWMO) & `Ztso`:** `m-store` relies on RVWMO's relaxed memory model to allow highly optimized, out-of-order memory operations, while optionally supporting the `Ztso` (Total Store Ordering) extension for stricter compatibility.
*   **`Zawrs` (Wait-on-Reservation-Set):** To eliminate the massive power waste of traditional software spin-loops, `m-store` utilizes the `Zawrs` extension to pause the hardware thread until a specific memory reservation is invalidated by another core.
*   **`Zihintntl` (Non-Temporal Locality):** During heavy MoE weight streaming or checkpointing, `m-store` issues `Zihintntl` hints to tell the hardware to bypass the cache, successfully mitigating L1/L2 cache pollution and preserving active working sets.
*   **The MLIR / LLVM Continuum:** To target these specialized hardware primitives, `m-store` relies on LLVM and MLIR as the unifying compiler infrastructure. This continuum bridges high-level domain-specific languages (like Triton for tensor math) directly to bare-metal hardware execution paths.
