# Module 03 — Storage, Networking & Data Pipelines (Chapter 3)

> GPU များ ဘယ်လောက် မြန်မြန် — data မမီလျှင် အလကား။ ဤ module တွင်
> storage architecture, high-speed network, bottleneck ရှာနည်းနှင့် ETL pipeline ဒီဇိုင်းကို ရှင်းပြပါမည်။

---

## 3.0 ဘာကြောင့် data movement က အရေးကြီးသလဲ (Background)

training job တစ်ခုက GPU ပေါ် မလာမီ data ကို storage → network → RAM → GPU memory
ဖြတ်သန်း ရပါသည် — တစ်ဆင့်နှေးပါက GPU က **စောင့်နေ**ရပြီး util ကျပါသည်။

```
storage → [network] → host RAM → [PCIe] → GPU memory → compute
   ↑ နှေးပါက တစ်ခုလုံး နှေး (GPU util ကျ)
```

## 3.1 Storage Architectures — အမျိုးအစား ၃ မျိုး ⭐

| အမျိုးအစား | ပုံစံ | ဥပမာ | AI အတွက် သုံးရာ |
|---|---|---|---|
| **Block** | raw device | NVMe, SAN | local scratch, DB |
| **File** | hierarchy | NFS, Lustre, GPFS | shared training data |
| **Object** | key-value | S3, MinIO | dataset archive, checkpoint |

### AI အတွက် ပုံမှန် ဒီဇိုင်း ⭐
```
┌─────────────────────────────────────────┐
│ Object storage (S3)   — dataset source  │
│ Parallel FS (Lustre)  — training read   │ ← အမြန်ဆုံး shared
│ Local NVMe            — scratch/cache   │ ← အနီးဆုံး
└─────────────────────────────────────────┘
```
- **Parallel FS** (Lustre/GPFS) — node ရာပေါင်းများ တပြိုင်နက် ဖတ်နိုင် — training အဓိက
- **Local NVMe** — checkpoint/intermediate အတွက် အမြန်ဆုံး
- **Object storage** — ပမာဏ ကြီး၊ ဈေးသက်သာ (versioning ပါ)

## 3.2 High-Speed Networking ⭐

```
intra-node:   NVLink (GPU↔GPU)        900GB/s အဆင့်
inter-node:   InfiniBand (NDR 400G) / RoCE
storage:      NVMe-oF, 200/400GbE
```

- **InfiniBand** — RDMA (kernel bypass), latency နည်း — HPC/AI စံ
- **RoCE** — Ethernet ပေါ် RDMA — datacenter ရှိပြီးသား fabric သုံးရာ
- **GPUDirect** — NIC/storage က GPU memory သို့ တိုက်ရိုက် (CPU/RAM ကျော်) —
  **GPUDirect Storage (GDS)** = storage → GPU တိုက်ရိုက် ⭐

စာမေးပွဲ trap — "GPU ကြား bandwidth" = NVLink၊ "node ကြား" = InfiniBand/RoCE။

## 3.3 Data Movement Bottlenecks & Optimization ⭐

### ဘယ်လို ရှာမလဲ
```
GPU util နည်းနေလျှင် —
① nvidia-smi (util/mem) ကြည့်
② CPU util (dataloader က CPU ပြည့်နေလား)
③ storage throughput (iostat/benchmark)
④ network (bandwidth/error)
```

### Optimization နည်းလမ်းများ ⭐
- **Prefetch/async loading** — GPU မလိုအပ်မီ data ကြိုတင် ဆောင်ရွက်
- **Parallel data loader** (PyTorch DataLoader workers)
- **Cached local NVMe** — hot dataset ကို node အတွင်း သိမ်း
- **GPUDirect Storage** — host RAM copy ဖယ်
- **DALI** — decode/augment ကို GPU ပေါ်
- **Mixed precision (FP16/BF16)** — data ပမာဏ လျှော့

## 3.4 AI Data Pipeline Design (ETL) ⭐

```
Extract        →    Transform       →    Load
(S3/object)        (clean, tokenize,     (training FS /
                    augment, format)      GPU-ready format)
```

### Production example လက်တွေ့ စီမံခန့်ခွဲမှု ⭐
- **Validation** — data quality/schema စစ် (corrupt/missing ဖယ်)
- **Versioning** — dataset version သတ်မှတ် (reproducibility)
- **Streaming** — တစ်ခါတည်း အကုန် မထည့်ဘဲ batch အလိုက် (Petabyte-scale)
- **Monitoring** — pipeline throughput/ error rate တိုင်းတာ

⚠️ pipeline က မတည်ငြိမ်လျှင် training ပြန်စရခြင်း/GPU idle ဖြစ်ရခြင်း —
**data pipeline quality = training reliability** ဖြစ်ပါသည်။ ⭐

## 5. အခြား sources ရှင်းလင်းချက်

- **စာအုပ်** — "storage က ဒေတာ ပမာဏပေါ်၊ network က ပေါင်းချုပ် traffic ပေါ် မူတည်
  ဒီဇိုင်း ပြုလုပ်ရမည်" — one-size-fits-all မရှိ
- **NVIDIA GDS docs** — "CPU ကို data path ကနေ ဖယ်ခြင်း" ကို bottleneck ဖြေရှင်းရေး၏
  နောက်ဆုံး အဆင့်အဖြစ် ဖော်ပြ
- **DC-Lab-Sim-course Module 08** — GPU util နည်းခြင်း ခွဲခြားနည်း (လက်တွေ့ ဗားရှင်း)

## 6. Self-Check

1. Lustre က ဘယ် storage အမျိုးအစားလဲ?
2. GPUDirect Storage က ဘာကို ဖယ်ပေးသလဲ?
3. ETL ၃ အဆင့်?

**အဖြေ** — 1) parallel file (shared) · 2) host RAM copy (storage→GPU တိုက်ရိုက်) · 3) Extract, Transform, Load

## 7. References & Further Reading

- GPUDirect Storage: https://docs.nvidia.com/gpudirect-storage/
- Magnum IO: https://developer.nvidia.com/magnum-io
- SuperPOD Storage: https://docs.nvidia.com/dgx-superpod/reference-architecture-scalable-infrastructure-h100/latest/storage-architecture.html
- NCCL: https://docs.nvidia.com/multi-node-nvlink-systems/multi-node-tuning-guide/nccl.html
- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135/3
