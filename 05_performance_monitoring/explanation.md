# Module 05 — Performance Optimization & Monitoring (Chapter 5)

> bottleneck ရှာနည်း — Nsight Systems, Nsight Compute, framework profiler, GPU telemetry —
> TensorRT optimization နှင့် measurement-led tuning loop ကို အပြည့် ရှင်းပြပါမည်။

---

## 5.0 Performance Philosophy — "တိုင်းပြီးမှ ပြင်" ⭐

စာအုပ်၏ အဓိက ဒဿန — **optimize မလုပ်ခင် တိုင်းတာပါ** (measure first) —
ခန့်မှန်းပြီး ပြင်ခြင်းက အချိန် ဆုံးရှုံးစေပါသည်။

```
measure → locate bottleneck → hypothesis → change ONE thing → re-measure
     ↑                                                          ↓
     └────────────────── iterate ←─────────────────────────────┘
```

## 5.1 Profiling Tools — ဘာကို ဘာနှင့် မြင်ရသလဲ ⭐⭐

| Tool | scope | မြင်ရမှု | ဘယ်အခါ |
|---|---|---|---|
| **Nsight Systems** (`nsys`) | system-wide timeline | CPU/GPU/kernel/memory/network အချိန်ဇယား | ဘယ်အဆင့် နှေးသလဲ ရှာ |
| **Nsight Compute** (`ncu`) | GPU kernel အသေးစိတ် | kernel occupancy, memory, instruction | kernel တစ်ခု ဘာကြောင့် နှေး |
| Framework profiler (PyTorch profiler) | framework op | op-level timing, memory | model ကုဒ် အဆင့် |
| DCGM / nvidia-smi | GPU state | util, temp, power, ECC | အခြေအနေ အမြဲ ကြည့် |

```
timeline မြင်ချင် → nsys     kernel အသေးစိတ် → ncu
op အဆင့်           → PyTorch profiler   state အမြဲ   → DCGM
```

### နမူနာ usage ⭐
```bash
nsys profile -o report python train.py          # system timeline
ncu --set full python train.py                   # kernel detail
ncu --kernel-name attention python train.py      # kernel ရွေး
```

## 5.2 Bottleneck အမျိုးအစား ၄ မျိုး ⭐⭐

| Bottleneck | လက္ခဏာ | ဖြေရှင်းနည်း |
|---|---|---|
| **Compute** | GPU util မြင့်၊ kernel ကြာ | kernel optimize, TensorRT, mixed precision |
| **Memory** | HBM bandwidth ပြည့် / OOM | memory layout, checkpointing, batch ချိန် |
| **Storage/Data** | GPU စောင့် (util နည်း) | prefetch, cache, GDS, DALI |
| **Network** | multi-node မှာ နှေး | NCCL tuning, fabric check, GPUDirect |

timeline တစ်ခုထဲတွင် — kernel များ ဆက်တိုက် = compute · gap ကြီး = data/network စောင့် —
Nsight Systems က ဤ gap ကို မြင်ရစေပါသည်။

## 5.3 TensorRT Optimization ⭐

TensorRT က trained model ကို inference အတွက် optimize ပြုလုပ်ပါသည် —
- **layer fusion** — op များ ပေါင်း (kernel လျှော့)
- **precision calibration** — FP32→FP16/INT8 (accuracy ထိန်းရင်း မြန်)
- **kernel auto-tuning** — GPU generation အလိုက် အကောင်းဆုံး kernel ရွေး

```
PyTorch model → export (ONNX) → TensorRT engine → Triton serve
                     optimize                (Module 08)
```

⚠️ optimize ပြီးလျှင် **accuracy ပြန်စစ်**ပါ — INT8 quantization က accuracy ကျစေနိုင်။

## 5.4 Measurement-Led Tuning Loop ⭐

```
① baseline တိုင်း (metric သတ်မှတ်: latency, throughput, GPU util)
② profile (nsys/ncu) → အကြီးဆုံး gap ရွေး
③ change တစ်ခုသာ ပြုလုပ်
④ re-measure → တိုးတက်မှု ရှိ/မရှိ
⑤ ရှိလျှင် keep; မရှိလျှင် revert → နောက် bottleneck
```
⚠️ change အများ တစ်ပြိုင်နက် ပြုလုပ်လျှင် ဘာက အလုပ်လုပ်သလဲ မသိနိုင် — **single variable** စည်းကမ်း။

## 5.5 အခြား sources ရှင်းလင်းချက်

- **စာအုပ်** — "profiling က အနုပညာ မဟုတ် — **စနစ်တကျ workflow**" —
  tool တစ်ခုတည်း မဟုတ်ဘဲ scope အလိုက် tool ပေါင်းစပ် အသုံးပြုရန်
- **NVIDIA Nsight docs** — Systems (timeline) နှင့် Compute (kernel) ကို
  "top-down" ချဉ်းကပ်မှု အဖြစ် ပေါင်းစပ် အသုံးပြုရန် အကြံပြု

## 6. Self-Check

1. timeline gap ကြီး မြင်ရပါက ဘာ သံသယ ရှိရမလဲ?
2. kernel တစ်ခု အသေးစိတ် ရှာရန် tool?
3. tuning loop တွင် change ဘာကြောင့် တစ်ခုသာ?

**အဖြေ** — 1) data/network စောင့်နေ (util နည်း) · 2) Nsight Compute · 3) variable ထိန်း — cause/effect သိရန်

## 7. References & Further Reading

- Nsight Systems: https://developer.nvidia.com/nsight-systems
- Nsight Compute: https://developer.nvidia.com/nsight-compute
- TensorRT: https://developer.nvidia.com/tensorrt
- PyTorch profiler: https://pytorch.org/tutorials/intermediate/tensorboard_profiler_tutorial.html
- DCGM: https://docs.nvidia.com/datacenter/dcgm/
