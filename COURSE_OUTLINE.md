# COURSE OUTLINE — Designing NVIDIA AI Infrastructure

> 4 modules (Chapters 1-4) · စာအုပ် structure အတိုင်း · NCP-AII domains နှင့် ပေါင်းစပ်

---

## Module 01 — AI Infrastructure Foundations (Chapter 1)
- **1.1 Introduction to AI infrastructure design** — layered stack (compute/storage/network/software/orchestration); design for production: reliability, scalability, observability, security; training vs inference requirements
- **1.2 The role of GPUs in AI workloads** — parallel matrix math; cores/memory bandwidth; framework support (CUDA, PyTorch, TensorFlow, JAX); NVIDIA GPU lineup (RTX → A100 → H100 → B200 → Jetson)
- **1.3 CPU vs GPU vs DPU architectures** — three-chip model; in practice (BlueField offload)
- **1.4 GPU acceleration for AI/ML pipelines** — data stages (ETL/preprocess); training, tuning (HPO), inference acceleration
- **1.5 NVIDIA ecosystem overview** — software stack (CUDA/cuDNN/cuBLAS/TensorRT/Triton/NeMo/RAPIDS); deployment (NGC, DOCA, GPU Operator, DCGM)

## Module 02 — GPU Resource Management & Virtualization (Chapter 2)
- **2.1 MIG configuration** — how MIG works (SM/memory slicing); profiles; configure & monitor (nvidia-smi mig)
- **2.2 GPU sharing and isolation** — time-slicing, MPS, memory management, scheduling
- **2.3 vGPU** — hypervisor-mediated; requirements, use cases (VDI), management/license
- **2.4 GPU workload scheduling with Kubernetes** — device plugin, `nvidia.com/gpu`, monitoring (DCGM exporter), best practices (quotas, placement, MIG strategies)

## Module 03 — Storage, Networking & Data Pipelines (Chapter 3)
- **3.1 Storage architectures** — block/file/object; parallel FS; local NVMe/scratch; AI requirements
- **3.2 High-speed networking** — InfiniBand, RoCE, NVLink; GPUDirect; bandwidth/latency
- **3.3 Data movement bottlenecks** — GPU starvation; dataloader, prefetch; GPUDirect Storage
- **3.4 AI data pipeline design** — ETL stage; production example (streaming, versioning, validation)

## Module 04 — Cluster Orchestration & Scalability (Chapter 4)
- **4.1 Kubernetes for GPU workloads** — enabling GPU support; requesting resources
- **4.2 Helm, operators, autoscaling** — Helm charts; GPU Operator; cluster autoscaler
- **4.3 Slurm, Kubeflow, MLflow integration** — scheduler vs pipeline vs experiment tracking
- **4.4 Cluster topologies** — deployment model ၃ မျိုး: On-premises, Cloud-native, Hybrid (cost/scale/control နှိုင်းယှဉ်)

## Module 05 — Performance Optimization & Monitoring (Chapter 5)
- Nsight Systems/Compute, framework profilers, GPU telemetry — bottleneck ၄ မျိုး (compute/memory/storage/network)
- TensorRT optimization; measurement-led tuning loop

## Module 06 — Security, Compliance & Data Governance (Chapter 6)
- Layered controls: platform trust, isolation, image verification, encryption, K8s RBAC, policy enforcement
- GDPR / HIPAA / FedRAMP — obligations နှင့် evidence

## Module 07 — Edge AI Infrastructure & Integration (Chapter 7)
- Edge vs cloud; Jetson/Orin platforms; federated learning vs distributed inference
- Smart city / retail / industrial IoT use cases

## Module 08 — NGC, Triton Inference Server & Deployment (Chapter 8)
- NGC containers/models/SDKs; Triton model repository, backends, schedulers, ensembles, APIs
- Load balancing, autoscaling, redundancy, failover patterns

## Module 09 — Real-World AI Infrastructure & Enterprise Workflows (Chapter 9)
- AI supercomputer case study; multi-tenant healthcare platform
- End-to-end lifecycle (data→training→deployment→monitoring→feedback loop)
