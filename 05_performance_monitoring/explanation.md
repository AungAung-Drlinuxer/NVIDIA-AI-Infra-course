# Module 05 — Performance Optimization & Monitoring (Chapter 5)

"training က ဘာကြောင့် နှေးတာလဲ" ဆိုသော မေးခွန်းက GPU engineer တစ်ယောက်
အမြဲ ရင်ဆိုင်ရမည့် မေးခွန်း ဖြစ်ပါသည်။ ခန့်မှန်းပြီး ပြင်ဆင်ခြင်းထက်
**တိုင်းတာပြီးမှ ပြင်ဆင်ခြင်း** က အချိန် ပိုသက်သာပြီး ရလဒ် ပိုတိကျပါသည်။
ဤ chapter သည် profiling tool များ၊ bottleneck ခွဲခြားနည်း၊ TensorRT optimization နှင့်
measurement-led tuning loop ကို ဖုံးလွှမ်းပါမည်။

## Tuning loop — ဘယ်လို ချဉ်းကပ်မလဲ

```
measure → locate bottleneck → hypothesis → ONE change → re-measure
   ↑                                                       ↓
   └──────────────────── iterate ←─────────────────────────┘
```

change ကို တစ်ကြိမ်တွင် တစ်ခုသာ ပြုလုပ်ပါ — တစ်ပြိုင်နက် ပြောင်းပါက
မည်သည့် ပြောင်းလဲမှုက အလုပ်လုပ်သလဲ ခွဲခြား၍ မရနိုင်ပါ။

## Profiling tool များ — ဘာကို ဘာနှင့် မြင်ရသလဲ

| Tool | scope | မြင်ရမှု | ဘယ်အခါ သုံးမလဲ |
|---|---|---|---|
| **Nsight Systems** (`nsys`) | system-wide timeline | CPU/GPU/kernel/memory/network အချိန်ဇယား | ဘယ်အဆင့်က အကြာဆုံးလဲ ရှာ |
| **Nsight Compute** (`ncu`) | GPU kernel အသေးစိတ် | occupancy, memory access, instruction | kernel တစ်ခု ဘာကြောင့် နှေးလဲ |
| Framework profiler (PyTorch profiler) | framework op | op-level timing/memory | model ကုဒ် အဆင့် |
| DCGM / `nvidia-smi` | GPU state | utilization, temperature, power, ECC | အခြေအနေ အမြဲ ကြည့် |

```bash
# system-wide timeline ဖမ်း — report ဖိုင် ထွက်လာမည်
nsys profile -o report python train.py

# kernel အသေးစိတ် ဖမ်း — GPU model ပေါ် မူတည် ကြာနိုင်
ncu --set full python train.py

# kernel အမည်ဖြင့် စစ်ထုတ်
ncu --kernel-name attention python train.py
```

⚠️ `nsys`/`ncu` command option များက tool version ပေါ် မူတည်၍ ပြောင်းလဲနိုင်ပါသည် —
`--help` သို့ version documentation ကို ကြည့်ပါ။ ဥပမာများက illustrative ဖြစ်ပါသည်။

## Bottleneck အမျိုးအစား ၄ မျိုး

| အမျိုးအစား | လက္ခဏာ | ဖြေရှင်းနည်း |
|---|---|---|
| **Compute** | GPU utilization မြင့်၊ kernel များ ကြာ | kernel optimize, TensorRT, mixed precision |
| **Memory** | bandwidth ပြည့် / OOM | memory layout, gradient checkpointing, batch ချိန် |
| **Storage/Data** | GPU စောင့် (util နည်း, gap ကြီး) | prefetch, cache, GPUDirect Storage, DALI |
| **Network** | multi-node တွင် ပိုနှေး | NCCL tuning, fabric link check, GPUDirect |

timeline တွင် kernel များ ဆက်တိုက် ပြည့်နေပါက compute၊ gap ကြီးပါက
data/network စောင့်နေခြင်း — Nsight Systems က ဤ gap ကို မြင်ရစေပါသည်။

## TensorRT optimization

TensorRT က trained model ကို inference အတွက် optimize ပြုလုပ်ပါသည် —

- **Layer fusion** — op များ ပေါင်း၍ kernel အရေအတွက် လျှော့ချခြင်း
- **Precision calibration** — FP32 → FP16/INT8 (accuracy ထိန်းရင်း မြန်အောင်)
- **Kernel auto-tuning** — target GPU generation အလိုက် kernel ရွေးချယ်ပေးခြင်း

```
PyTorch model → ONNX export → TensorRT engine → Triton serve
```

⚠️ quantization (INT8) က accuracy ကျစေနိုင်သောကြောင့် optimize ပြီးတိုင်း
accuracy ကို validation set ဖြင့် ပြန်စစ်ပါ။

## လုပ်ငန်းခွင် သတိပေးချက်များ

- utilization ဂဏန်းတစ်ခုတည်းကို ကြည့်ပြီး bottleneck မဆုံးဖြတ်ပါနှင့် —
  utilization မြင့်ရုံနှင့် efficient မဟုတ်ပါ (memory-bound ဖြစ်နိုင်)
- profile run က production run ထက် overhead ပိုပါသည် — နှိုင်းယှဉ်ပါက အခြေအနေတူ
  run များဖြင့်သာ နှိုင်းယှဉ်ပါ
- benchmark ဂဏန်းများက hardware/config ပေါ် မူတည်ပါသည် — ဤနေရာတွင်
  ဂဏန်း မဖော်ပြထားပါ၊ မိမိ environment တွင်သာ တိုင်းတာပါ

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. timeline တွင် gap ကြီး မြင်ရပါက ဘာ သံသယ ရှိရမလဲ။
2. kernel တစ်ခု၏ အသေးစိတ်ကို ရှာရန် tool ဘာလဲ။
3. tuning loop တွင် change ကို ဘာကြောင့် တစ်ခုသာ ပြုလုပ်ရသလဲ။
4. TensorRT optimize ပြီးပါက ဘာ ပြန်စစ်ရမလဲ။

**အဖြေ** — 1) data/network စောင့်နေခြင်း · 2) Nsight Compute · 3) cause/effect ခွဲခြားနိုင်ရန် · 4) accuracy (quantization ကျမှု)

## Technical review notes

- `nsys`/`ncu` option များနှင့် output format သည် tool version ပေါ် မူတည်ပါသည် — ပြသထားသော command များက illustrative ဖြစ်ပြီး မိမိ version ၏ documentation ဖြင့် အတည်ပြုပါ။
- TensorRT precision/accuracy trade-off သည် model နှင့် dataset ပေါ် မူတည်ပါသည် (needs verification per model)။

## References

- Nsight Systems: https://developer.nvidia.com/nsight-systems
- Nsight Compute: https://developer.nvidia.com/nsight-compute
- TensorRT: https://developer.nvidia.com/tensorrt
- PyTorch profiler: https://pytorch.org/tutorials/intermediate/tensorboard_profiler_tutorial.html
- DCGM: https://docs.nvidia.com/datacenter/dcgm/
