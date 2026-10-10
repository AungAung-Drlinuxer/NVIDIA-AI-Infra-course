# Module 01 — AI Infrastructure Foundations (Chapter 1)

AI model တစ်ခု လေ့ကျင့်ပြီး production တွင် အလုပ်လုပ်စေရန် hardware ဝယ်ရုံနှင့် မပြီးပါ။
compute, storage, networking, software, orchestration ဆိုသော အလွှာ ၅ ခု ပေါင်းစပ်ပြီး
ကုန်ကျစရိတ်၊ စွမ်းဆောင်ရည်၊ လုံခြုံရေး ဟန်ချက်ညီအောင် ဒီဇိုင်းဆွဲရပါမည်။
ဤ chapter သည် ထိုအလွှာများကို မိတ်ဆက်ပြီး CPU, GPU, DPU တို့၏ အခန်းကဏ္ဍနှင့်
NVIDIA software ecosystem ကို ခြုံငုံ မိတ်ဆက်ပါမည်။

## AI platform ဆိုတာ ဘယ်လို အလွှာများနှင့် ဖွဲ့စည်းထားသလဲ

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

အလွှာတစ်ခုချင်းစီသည် သီးခြား ရွေးချယ်မှု လိုအပ်ပြီး အလွှာချင်း interface က
စွမ်းဆောင်ရည်ကို သတ်မှတ်ပါသည် — ဥပမာ storage နှေးပါက GPU ဘယ်လောက် ကောင်းကောင်း
utilization ကျလာပါမည်။

## Production အတွက် ဒီဇိုင်းဆွဲရာတွင် ထည့်စဉ်းစားရမည့် အချက် ၄ ခု

| အချက် | ဆိုလိုရင်း | လက်တွေ့ ဆောင်ရွက်ချက် |
|---|---|---|
| Reliability | hardware ပျက်မှု ဖြစ်လာမည်ကို ကြိုတင် လက်ခံ | redundancy, failover, burn-in test |
| Scalability | node ထပ်ထည့်လျှင် စွမ်းဆောင်ရည် တန်းတန်းတက် | scale-up vs scale-out ရွေးချယ်မှု |
| Observability | မတိုင်းတာနိုင်လျှင် မစီမံနိုင် | DCGM, Prometheus, Grafana |
| Security | multi-tenant အသုံးပြုသူ ဒေတာ ခွဲခြား | isolation, RBAC, audit (Module 06) |

ထို့အပြင် **training** နှင့် **inference** က requirement မတူပါ။ training သည်
ရက်ပေါင်းများစွာ ဆက်တိုက် run ပြီး bandwidth အမြင့် လိုသည်။ inference က latency နည်းပြီး
စဉ်ဆက်မပြတ် ရရှိနေရမည် — ဒီဇိုင်းက ၂ မျိုးလုံးကို ထည့်စဉ်းစားရပါမည်။

## CPU, GPU, DPU — တစ်ခုချင်း ဘာအတွက် တာဝန်ယူသလဲ

| Chip | ပုံမှန် တာဝန် | ဘာကြောင့် ထိုအလုပ်အတွက် သင့်သလဲ |
|---|---|---|
| **CPU** | OS, control plane, data prep | logic များ၊ branch များသော အလုပ် |
| **GPU** | model training/inference | matrix math ကို ကြီးမားစွာ parallel လုပ် |
| **DPU** (BlueField) | network, storage, security offload | infrastructure အလုပ်ကို သီးခြား chip ဖြင့် ကိုင် |

DPU က networking/storage/security ကို CPU မှ ဖယ်ရှားပေးသောကြောင့် CPU သည်
workload အတွက် ပိုလွတ်ပါသည်။ တစ်ချိန်တည်းတွင် infrastructure software ကို
tenant workload မှ သီးခြားဖြစ်စေပြီး လုံခြုံရေးအတွက်လည်း အထောက်အကူ ဖြစ်ပါသည်။

## GPU က AI workload အတွက် ဘာကြောင့် ပိုသင့်တော်သလဲ

AI ရဲ့ အလုပ် အများစု (convolution, attention, dense layer) သည် matrix/vector
operation ဖြစ်ပြီး အပိုင်းပေါင်းများစွာ ခွဲလုပ်နိုင်ပါသည်။ CPU က core အနည်းငယ်
ကြီးကြီးဖြင့် sequential အလုပ်အတွက် ပြင်ဆင်ထားခြင်းဖြစ်ပြီး GPU က core သေးသေး
ထောင်ချီဖြင့် parallel throughput အတွက် တည်ဆောက်ထားခြင်း ဖြစ်ပါသည်။

- **CUDA cores** — အထွေထွေ parallel math
- **Tensor Cores** — matrix multiply-accumulate အတွက် အထူး unit (FP16/BF16/FP8 စသည်)
- **Memory bandwidth** — HBM က CPU ၏ DDR ထက် များစွာ မြန်သောကြောင့် data ကို
  compute unit က မစောင့်ရ

PyTorch, TensorFlow, JAX စသည့် framework အများစုက CUDA ပေါ်တွင် အလုပ်လုပ်သောကြောင့်
NVIDIA GPU က ecosystem အပြည့်အစုံ ရရှိပါသည်။

GPU lineup ကို အသုံးပြုမှု အလိုက် ခွဲနိုင်ပါသည် — desktop development အတွက် RTX series၊
data center training/inference အတွက် A-series (Ampere), H-series (Hopper), B-series
(Blackwell)၊ edge device အတွက် Jetson series (Module 07)။ model တစ်ခုချင်း၏ core count,
memory ပမာဏ၊ bandwidth များက generation နှင့် form factor ပေါ် မူတည်သောကြောင့်
ဝယ်ယူမီ တရားဝင် specification ကို စစ်ဆေးသင့်ပါသည်။

## AI/ML pipeline တစ်လျှောက် GPU acceleration

```
data → [Extract] → [Transform/Preprocess] → [Train] → [Tune] → [Inference]
        I/O          GPU (DALI/RAPIDS)       GPU ပင်မ   parallel  GPU (TensorRT/Triton)
```

- **Data stage** — image decode, augmentation, tokenization ကို GPU ပေါ်တွင် လုပ်ပါက
  CPU bottleneck လျှော့ချနိုင်ပါသည်
- **Training** — GPU ၏ ပင်မ အလုပ်
- **Tuning (HPO)** — experiment များကို GPU အများဖြင့် တပြိုင်နက် run
- **Inference** — TensorRT ဖြင့် optimize ပြီး Triton ဖြင့် serve (Module 08)

data pipeline မမြန်ပါက GPU က စောင့်နေရပါမည် — GPU utilization နည်းခြင်း၏
အကြောင်းရင်း အဖြစ်များဆုံးက GPU ပြဿနာ မဟုတ်ဘဲ data supply ပြဿနာ ဖြစ်လေ့ရှိပါသည်။

## NVIDIA software stack နှင့် deployment component များ

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

| Component | အခန်းကဏ္ဍ |
|---|---|
| **NGC** | container/model catalog (validated image များ) |
| **GPU Operator** | Kubernetes ထဲ GPU stack ကို အလိုအလျောက် တပ်ဆင်/စီမံ |
| **DCGM** | GPU telemetry နှင့် diagnostics |
| **DOCA** | DPU platform SDK |
| **Triton** | inference serving (Module 08) |

## လုပ်ငန်းခွင် သတိပေးချက်များ

- GPU တစ်ခုတည်း ဝယ်ခြင်းက ပြဿနာ မဖြေရှင်းပါ — storage/network/pipeline အားနည်းပါက
  GPU က အလဟ့် ဖြစ်နေပါမည်။
- three-chip model ကို အမှတ်အသား အဖြစ်သာ မဟုတ်ဘဲ — ဘယ် component ပျက်လျှင်
  ဘယ်အလွှာ ထိခိုက်မလဲ ဆိုသော ခွဲခြားမှု အဖြစ် အသုံးပြုပါ။
- framework "CUDA-enabled" ဆိုခြင်းက CUDA version တိကျစွာ ကိုက်ညီမှု လိုအပ်ပါသည် —
  version matrix ကို အမြဲ စစ်ဆေးပါ။

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. AI infrastructure အလွှာ ၅ ခု ကို အမည်ပေးပါ။
2. GPU က CPU ထက် matrix math ပိုမြန်ရသော အကြောင်းရင်း ၂ ခု ပြောပါ။
3. DPU က မည်သည့် အလုပ် ၃ မျိုးကို offload လုပ်သလဲ။
4. GPU utilization နည်းနေလျှင် ပထမ သံသယ ရှိသင့်သည့် အကြောင်းရင်း ဘာလဲ။

**အဖြေ** — 1) storage, networking, compute, software, orchestration · 2) parallel core အများအပြားနှင့် memory bandwidth မြင့်မှု · 3) networking, storage, security · 4) data supply (dataloader/preprocessing) မမီခြင်း

## Technical review notes

- GPU lineup specification (core count, HBM bandwidth) များက generation နှင့် SKU ပေါ် မူတည်သောကြောင့် ဤတွင် ဂဏန်း မဖော်ပြထားပါ — လိုအပ်ပါက သက်ဆိုင်ရာ GPU ၏ datasheet မှ အတည်ပြုပါ။
- CUDA framework compatibility သည် version pair ပေါ် မူတည်ပါသည် (needs verification per target stack)။

## References

- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135
- CUDA Toolkit: https://developer.nvidia.com/cuda-toolkit
- NGC docs: https://docs.nvidia.com/ngc/latest/ngc-user-guide.html
- TensorRT: https://developer.nvidia.com/tensorrt
- Triton Inference Server: https://developer.nvidia.com/triton-inference-server
