# Module 01 — AI Infrastructure Foundations (Chapter 1)

> စာအုပ် Chapter 1 အတိုင်း — AI infrastructure ဒီဇိုင်း အခြေခံ၊ GPU ၏ အခန်းကဏ္ဍ၊
> CPU/GPU/DPU သုံးခု၊ pipeline acceleration နှင့် NVIDIA ecosystem ကို အပြည့် ရှင်းပြပါမည်။

---

## 1.1 AI Infrastructure Design မိတ်ဆက် (Background)

AI infrastructure ဆိုတာ hardware စုပေါင်းမှု မဟုတ်ပါ — **compute, storage, networking,
software, orchestration** အလွှာ ၅ ခု ပေါင်းစပ် ဒီဇိုင်း ဖြစ်ပါသည်။

```
┌───────────────────────────────────────────────────────┐
│ ⑤ Orchestration   Kubernetes · Slurm · Run:ai         │
├───────────────────────────────────────────────────────┤
│ ④ Software        CUDA · frameworks · NGC · Triton    │
├───────────────────────────────────────────────────────┤
│ ③ Compute         CPU · GPU · DPU (node အတွင်း)       │
├───────────────────────────────────────────────────────┤
│ ② Networking      InfiniBand/RoCE · NVLink            │
├───────────────────────────────────────────────────────┤
│ ① Storage         parallel FS · object · local NVMe   │
└───────────────────────────────────────────────────────┘
```

### Design for Production — စိတ်ကူး ၄ ခု ⭐

စာအုပ်က production ဒီဇိုင်းအတွက် အောက်ပါ ၄ ခုကို အလေးပေးပါသည် —

1. **Reliability** — hardware ပျက်မှု ဖြစ်နိုင်ကြောင်း ကြိုတင် လက်ခံ — redundancy, failover, burn-in
2. **Scalability** — node ထပ်ထည့်လျှင် performance တန်းတန်းတက် — scale-up vs scale-out
3. **Observability** — DCGM/Prometheus/Grafana — တိုင်းတာ၍ မမြင်နိုင်လျှင် မစီမံနိုင်
4. **Security** — multi-tenant isolation, access control, supply chain

**Training** (job ကြီး၊ ရက်ပေါင်းများ၊ bandwidth မြင့်) နှင့် **Inference** (latency နည်း၊
စဉ်ဆက်မပြတ်၊ uptime) က requirement မတူ — ဒီဇိုင်းက ၂ မျိုးလုံးကို ထည့်စဉ်းစားရပါမည်။ ⭐

## 1.2 GPU ၏ အခန်းကဏ္ဍ — ဘာကြောင့် CPU ထက် မြန်သလဲ ⭐

```
CPU (16 cores)                    GPU (A100: 6912 CUDA cores)
├─ အနည်းငယ် core ကြီးကြီး      ├─ core သေးသေး ထောင်ချီ
├─ branch ပြတ်/logic ကောင်း       ├─ matrix math များများ တပြိုင်နက်
└─ sequential အမြန်ဆုံး           └─ parallel throughput အမြင့်ဆုံး
```

AI ၏ အလုပ် အများစုက **matrix/vector operation** (convolution, attention) ဖြစ်ပြီး
ထိုအလုပ်က ထောင်ချီ အပိုင်း ခွဲလို့ ရပါသည် — GPU က ထို့ကြောင့် အနိုင်ရခြင်း ဖြစ်ပါသည်။

- **CUDA cores** — အထွေထွေ parallel math
- **Tensor Cores** — matrix multiply-accumulate (FP16/BF16/FP8) အထူး unit
- **Memory bandwidth** — HBM: A100 2TB/s vs CPU DDR4 ~50GB/s — data ကို မြန်မြန် ကျွေးနိုင်

### Framework support နှင့် GPU lineup

PyTorch/TensorFlow/JAX အားလုံးက **CUDA** ပေါ် တည်ဆောက်ထား — NVIDIA GPU သာ
ecosystem အပြည့် ရပါသည်။

| Lineup | သုံးရာ | ဥပမာ |
|---|---|---|
| RTX/GeForce | desktop dev, inference ငယ် | RTX 5090 |
| A-series (Ampere) | data center training/inference | A100, A30 |
| H-series (Hopper) | LLM training | H100, H200 |
| B-series (Blackwell) | next-gen training | B200, GB200 |
| Jetson | edge AI | Orin, Thor |

## 1.3 CPU vs GPU vs DPU — three-chip model ⭐

```
┌────────────────────────────────────────────────────┐
│                    AI Server                        │
│  ┌────────┐   ┌────────┐   ┌────────┐              │
│  │  CPU   │   │  GPU   │   │  DPU   │              │
│  │general │   │parallel│   │infra   │              │
│  │logic   │   │math    │   │offload │              │
│  └────────┘   └────────┘   └────────┘              │
│   OS/app       AI compute    network/storage/      │
│   control                    security             │
└────────────────────────────────────────────────────┘
```

- **CPU** — OS, control plane, data prep (logic များ)
- **GPU** — model training/inference (math များ)
- **DPU (BlueField)** — networking/storage/security ကို ကိုယ်ပိုင် processor ဖြင့် offload —
  CPU က workload အတွက် လွတ်ပြီး infrastructure က သီးခြား (isolation)

**practice ထဲ** — DGX တစ်လုံးတွင် ၃ ခုလုံး ပါဝင်ပြီး DPU က NIC/storage ကိုင်၊ CPU က job စီမံ၊
GPU က math လုပ်ပါသည်။

## 1.4 GPU Acceleration for AI/ML Pipelines ⭐

```
data → [Extract] → [Transform/Preprocess] → [Train] → [Tune] → [Inference]
         ↑ GPU:      ↑ GPU: RAPIDS,         ↑ GPU:    ↑ GPU:   ↑ GPU: TensorRT/
       I/O/storage   decode, augment        full      HPO      Triton
```

- **Data stages** — image decode, augmentation, tokenization ကို GPU (DALI/RAPIDS) ဖြင့်
  လုပ်ပါက CPU bottleneck လျှော့ချနိုင်
- **Training** — GPU ၏ ပင်မ အလုပ်
- **Tuning (HPO)** — experiment များ ကို parallel GPU များဖြင့် တပြိုင်နက်
- **Inference** — TensorRT (optimize) + Triton (serve) — latency/throughput ပြည့်မီ

⚠️ GPU ဝယ်ရုံနှင့် မပြီး — **data pipeline မမြန်လျှင် GPU က စောင့်နေ**မည် (Chapter 3 တွင် အပြည့်)။

## 1.5 NVIDIA Ecosystem Overview ⭐

### Software stack
```
┌────────────────────────────────────────┐
│ App     NeMo · RAPIDS · Clara          │
├────────────────────────────────────────┤
│ Serving TensorRT · Triton Inference    │
├────────────────────────────────────────┤
│ Train   PyTorch/TF/JAX (CUDA-enabled)  │
├────────────────────────────────────────┤
│ Library cuDNN · cuBLAS · NCCL · DALI   │
├────────────────────────────────────────┤
│ Core    CUDA Toolkit                   │
└────────────────────────────────────────┘
```

### Deployment/infrastructure components
| Component | ဘာလုပ် |
|---|---|
| **NGC** | container/model catalog |
| **GPU Operator** | K8s ထဲ GPU stack အလိုအလျောက် တပ် |
| **DCGM** | GPU telemetry/diagnostics |
| **DOCA** | DPU platform SDK |
| **Triton** | inference server |

![CUDA programming model](../images/cuda-programming-model.jpg)
*ပုံ — CUDA programming model (ရင်းမြစ်: NVIDIA Developer Blog)*

## 6. အခြား sources ရှင်းလင်းချက်

- **စာအုပ် (Designing NVIDIA AI Infrastructure)** — "infrastructure က model ထက်
  **workload fit** ကို ပိုဦးစားပေးရမည်" ဟူသော ဒီဇိုင်း ဒဿနကို အလေးပေး
- **NVIDIA docs** — three-chip model ကို "CPU=control, GPU=compute, DPU=infrastructure"
  အဖြစ် တည်နေရာ ၃ ခု ခွဲ ဖော်ပြ

## 7. Self-Check

1. AI infrastructure အလွှာ ၅ ခု?
2. GPU က CPU ထက် matrix math ပိုမြန်ရခြင်း အကြောင်းရင်း ၂ ခု?
3. DPU က ဘာတွေ offload လုပ်သလဲ?

**အဖြေ** — 1) storage/network/compute/software/orchestration · 2) core အများ (parallel) + memory bandwidth · 3) network/storage/security

## 8. References & Further Reading

- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135
- CUDA: https://developer.nvidia.com/cuda-toolkit
- NGC: https://docs.nvidia.com/ngc/latest/ngc-user-guide.html
- TensorRT: https://developer.nvidia.com/tensorrt · Triton: https://developer.nvidia.com/triton-inference-server
