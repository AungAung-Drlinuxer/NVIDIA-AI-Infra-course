# Module 03 — Storage, Networking & Data Pipelines (Chapter 3)

GPU cluster ရဲ့ စွမ်းဆောင်ရည်က GPU အပေါ်တွင်သာ မမူတည်ပါ — data က
GPU ဆီ ဘယ်လောက် မြန်မြန် ရောက်သလဲ ဆိုတာက တစ်ခါတရံ GPU ထက် ပိုအရေးကြီးပါသည်။
ဤ chapter သည် storage architecture ရွေးချယ်မှု၊ high-speed network ဖွဲ့စည်းပုံ၊
data bottleneck ရှာဖွေနည်းနှင့် ETL pipeline ဒီဇိုင်းကို ဖုံးလွှမ်းပါမည်။

## Data movement က ဘာကြောင့် အရေးကြီးသလဲ

training job တစ်ခုအတွက် data က အောက်ပါ လမ်းကြောင်း တစ်လျှောက် ဖြတ်သွားပါသည် —

```
storage → [network] → host RAM → [PCIe] → GPU memory → compute
```

တစ်နေရာတွင် နှေးပါက GPU က စောင့်နေရပြီး utilization ကျပါသည် —
ဆိုလိုသည်မှာ ဈေးကြီး GPU က အလဟ့် ဖြစ်နေပါသည်။

## Storage အမျိုးအစား ၃ မျိုးနှင့် AI cluster ဒီဇိုင်း

| အမျိုးအစား | ပုံစံ | ဥပမာ | AI တွင် အသုံးပြုပုံ |
|---|---|---|---|
| **Block** | raw device | local NVMe, SAN | scratch, database |
| **File** | directory hierarchy | NFS, Lustre, GPFS | shared training dataset |
| **Object** | key-value + metadata | S3, MinIO | dataset archive, checkpoint, versioning |

လက်တွေ့ cluster တစ်ခုတွင် သုံးခုလုံး ပေါင်းသုံးလေ့ ရှိပါသည် —

```
┌────────────────────────────────────────────────────┐
│ Object storage (S3)      — dataset source/archive  │
│ Parallel FS (Lustre/GPFS)— training read (shared)  │
│ Local NVMe               — cache/checkpoint (fast) │
└────────────────────────────────────────────────────┘
```

- **Parallel file system** က node ရာပေါင်းများ တပြိုင်နက် ဖတ်ခြင်းကို ကိုင်တွယ်နိုင်သောကြောင့်
  large-scale training အတွက် အဓိက ဖြစ်ပါသည်
- **Local NVMe** က အနီးဆုံး ဖြစ်သောကြောင့် hot cache နှင့် checkpoint အတွက် သင့်ပါသည်
- **Object storage** က ပမာဏ ကြီးပြီး ကုန်ကျစရိတ် သက်သာသည် — versioning ပါဝင်၍
  dataset ဗားရှင်း စီမံရာတွင် အသုံးဝင်ပါသည်

## High-speed network — ဘယ်လမ်းကြောင်းမှာ ဘာသုံးမလဲ

| လမ်းကြောင်း | နည်းပညာ | ဘာကြောင့် |
|---|---|---|
| GPU ↔ GPU (node အတွင်း) | **NVLink** | host PCIe ထက် bandwidth အမြင့် |
| node ↔ node | **InfiniBand** / **RoCE** | RDMA — CPU overhead နည်း, latency နည်း |
| node ↔ storage | high-speed Ethernet / NVMe-oF | storage traffic သီးခြား |

- **InfiniBand** (HDR 200G, NDR 400G စသည်) က RDMA (kernel bypass) ကြောင့်
  latency နည်းပြီး HPC/AI cluster ၏ အသုံးများဆုံး fabric ဖြစ်ပါသည်
- **RoCE** (RDMA over Converged Ethernet) က Ethernet fabric ရှိပြီးသား datacenter
  များအတွက် ရွေးချယ်စရာ ဖြစ်ပါသည်
- **GPUDirect** က NIC/storage က GPU memory သို့ တိုက်ရိုက် ရောက်စေပါသည် —
  **GPUDirect Storage** ဆိုပါက storage → GPU တိုက်ရိုက် ဖြစ်ပြီး host RAM copy ကို
  ဖယ်ရှားပေးပါသည်

ဗဟုသုတ — "GPU ကြား bandwidth" မေးပါက NVLink၊ "node ကြား" မေးပါက
InfiniBand/RoCE ဟု ခွဲခြား ဖြေလေ့ ရှိပါသည်။

## Bottleneck ရှာဖွေခြင်းနှင့် ဖြေရှင်းခြင်း

GPU utilization နည်းနေပါက အောက်ပါ အစဉ်အတိုင်း ခွဲခြား စစ်ဆေးပါ —

1. `nvidia-smi` — GPU state (utilization, memory) ကြည့်ပါ
2. CPU utilization — dataloader/preprocessing က CPU ပြည့်နေခြင်း ရှိ/မရှိ
3. storage throughput — read bandwidth လုံလောက်ခြင်း ရှိ/မရှိ
4. network bandwidth/error — fabric ပြဿနာ ရှိ/မရှိ

| တွေ့ရှိချက် | ဖြေရှင်းနည်း |
|---|---|
| GPU စောင့်နေ (util နည်း, gap ကြီး) | prefetch, parallel data loader, cache |
| preprocess က CPU ပြည့် | GPU-based decode (DALI), batch preprocess |
| storage read နှေး | local NVMe cache, GPUDirect Storage |
| multi-node မှာ ပိုနှေး | NCCL tuning, fabric link check |

အခြား optimization များ — mixed precision (FP16/BF16) ဖြင့် data/memory ပမာဏ
လျှော့ချခြင်း၊ asynchronous loading ဖြင့် GPU compute နှင့် data load overlap လုပ်ခြင်း။

## AI data pipeline (ETL) ဒီဇိုင်း

```
Extract        →    Transform        →    Load
(S3/object)         (clean, tokenize,     (parallel FS /
                     augment, format)      GPU-ready format)
```

production pipeline တစ်ခုတွင် ETL ထက် ပိုလိုအပ်ပါသည် —

- **Validation** — corrupt/missing data ကို ဝင်မလာစေရန် schema/quality check
- **Versioning** — dataset version သတ်မှတ်ခြင်းက experiment reproducibility အတွက် အရေးကြီးပါသည်
- **Streaming** — Petabyte-scale တွင် အကုန် load မလုပ်ဘဲ batch/streaming ဖြင့် ဖြည်းဖြည်း သုံးပါ
- **Monitoring** — pipeline throughput နှင့် error rate ကို တိုင်းတာပါ

pipeline မတည်ငြိမ်ပါက training ပြန်စရခြင်း၊ GPU idle ဖြစ်ရခြင်း ဆိုသည်မှာ
data pipeline quality က training reliability ဖြစ်ကြောင်း ပြပါသည်။

## လုပ်ငန်းခွင် သတိပေးချက်များ

- storage/network ဒီဇိုင်းကို "အမြန်ဆုံး" မဟုတ်ဘဲ **workload pattern နှင့် ကိုက်ညီမှု**ပေါ်
  ရွေးချယ်ပါ — training read pattern နှင့် archive pattern မတူပါ
- GPUDirect Storage သုံးပါက filesystem နှင့် driver support ကို အတည်ပြုပါ
  (version-dependent)
- fabric bandwidth ဂဏန်းများ (200G/400G...) က generation ပေါ် မူတည်ပါသည် —
  တပ်ဆင်မီ vendor spec မှ အတည်ပြုပါ

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. Lustre က storage အမျိုးအစား ဘာလဲ။
2. GPUDirect Storage က မည်သည့် copy ကို ဖယ်ရှားပေးသလဲ။
3. ETL အဆင့် ၃ ခု ကို ပြောပါ။
4. GPU utilization နည်းနေပါက ပထမဆုံး ဘာ စစ်မလဲ။

**အဖြေ** — 1) parallel file system (shared) · 2) host RAM copy (storage → GPU တိုက်ရိုက်) · 3) Extract, Transform, Load · 4) data supply (dataloader/throughput) ကို စစ်ဆေး

## Technical review notes

- fabric bandwidth (NVLink/IB/RoCE) ဂဏန်းများက generation နှင့် port config ပေါ် မူတည်ပါသည် — ဤတွင် အမျိုးအစား အဆင့်သာ ဖော်ပြထားပါသည်။
- GPUDirect Storage support သည် filesystem နှင့် software version ပေါ် မူတည်ပါသည် (needs verification per environment)။

## References

- GPUDirect Storage: https://docs.nvidia.com/gpudirect-storage/
- Magnum IO: https://developer.nvidia.com/magnum-io
- NCCL tuning: https://docs.nvidia.com/multi-node-nvlink-systems/multi-node-tuning-guide/nccl.html
- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135/3
