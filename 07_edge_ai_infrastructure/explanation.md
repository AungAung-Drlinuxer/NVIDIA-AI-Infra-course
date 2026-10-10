# Module 07 — Edge AI Infrastructure & Integration (Chapter 7)

AI အားလုံး datacenter မှာ မလည်ပတ်ပါ — ကားပေါ်၊ စက်ရုံထဲ၊ စတိုးဆိုင်မှာ
data ဖြစ်ပွားရာ နေရာ အနီးတွင် ချက်ချင်း ဆုံးဖြတ်ချက် ချရန် လိုအပ်ပါသည်။
ဤ chapter သည် edge AI ၏ လိုအပ်ချက်၊ edge/cloud ခွဲဝေမှု၊ NVIDIA Jetson
platform နှင့် federated learning ကဲ့သို့ နည်းပညာ များကို ရှင်းလင်းပါမည်။

## Edge AI က ဘာပြဿနာ ဖြေရှင်းပေးသလဲ

```
cloud AI:  sensor → [network] → cloud GPU → result → [network] → device
edge AI:   sensor → [edge device GPU] → result        (network မလို)
```

| အချက် | Edge | Cloud |
|---|---|---|
| Latency | မီလီစက္ကန့် အဆင့် (local) | network ပေါ် မူတည် |
| Connectivity | offline လုပ်နိုင် | အမြဲ ချိတ်ရန် လို |
| Compute | power ဖြင့် ကန့်သတ် | လက်တွေ့ အကန့်အသတ် မရှိ |
| Data privacy | ဒေတာ မထွက် | ပို့ရနိုင် |
| Management | device များ ဖြန့်ကျက် စီမံရ | စီမံရ လွယ် |

edge က cloud ၏ အစားထိုး မဟုတ်ပါ — workload ကို latency, privacy,
bandwidth လိုအပ်ချက် ပေါ် မူတည်၍ ခွဲဝေ ချထားခြင်း ဖြစ်ပါသည်။

## NVIDIA Jetson platform

```
┌─ Jetson Orin module ─────────────────────────┐
│ GPU (Ampere architecture) + CPU + memory + ISP│
│ power budget က module အလိုက် ကွာခြား         │
└───────────────────────────────────────────────┘
```

| Module | ပုံမှန် အသုံးပြုမှု |
|---|---|
| Orin Nano | camera, robot (power အနည်းဆုံး) |
| Orin NX | smart city, retail |
| AGX Orin | industrial, autonomous system |

- **JetPack SDK** တွင် CUDA, TensorRT, DeepStream (video AI) ပါဝင်ပါသည်
- datacenter နှင့် CUDA/TensorRT toolchain တူညီသောကြောင့် model ပြန်မရေးဘဲ
  optimize ပြီး ဖြန့်ချိနိုင်ပါသည်
- power/thermal ကန့်သတ်ချက် ရှိသောကြောင့် model ကို TensorRT/FP16 ဖြင့်
  optimize လုပ်ရန် ပုံမှန် လိုအပ်ပါသည်

⚠️ Jetson module တစ်ခုချင်း၏ power/memory spec များက product generation ပေါ်
မူတည်ပါသည် — ဝယ်ယူမီ တရားဝင် specification မှ အတည်ပြုပါ။

## Federated learning vs distributed inference

ဤ ၂ ခုကို မရောထွားပါနှင့် — တစ်ခုချင်းက မတူသော ပြဿနာကို ဖြေရှင်းပါသည်။

| | Federated Learning | Distributed Inference |
|---|---|---|
| ဘာလုပ် | **model ကို** data ရှိရာ ပို့ လေ့ကျင့် | **request ကို** inference node များ ဖြန့် |
| data | device ထဲမှ မထွက် (gradient ပဲ ပို့) | request/response ပို့ |
| ရည်ရွယ် | privacy-preserving training | serving load ဖြန့်ဝေ |

```
Federated:         edge (local train) → gradient → aggregate → model ပြန်ဖြန့်
Dist. inference:   request → load balancer → inference nodes → response
```

federated learning က **training ဘက်**၊ distributed inference က **serving ဘက်** ဖြစ်ပါသည်။

## Use case များ

| နယ်ပယ် | ဥပမာ | edge ဘာကြောင့် သင့်သလဲ |
|---|---|---|
| Smart city | traffic camera, public safety | latency + bandwidth |
| Retail | shelf monitoring, checkout | real-time + privacy |
| Industrial IoT | defect detection, predictive maintenance | downtime လျှော့ |

ပုံမှန် ဖွဲ့စည်းပုံ — edge device က preprocess/inference ကို ကိုင်တွယ်ပြီး
cloud က retrain/aggregate/fleet management ကို ကိုင်တွယ်ပါသည်။

## လုပ်ငန်းခွင် သတိပေးချက်များ

- edge deployment တွင် **fleet management** (update, monitoring) က model
  accuracy ထက် ပိုခက်ခဲတတ်ပါသည် — OTA update နှင့် rollback စီစဉ်ချက် ကြိုတင်ပါ
- federated learning တွင် gradient က privacy အပြည့် မပေးနိုင်ပါ (inference
  attack မျိုး ရှိနိုင်) — technique အဆင့် ပိုကြည့်ရန် လိုအပ်ပါသည်
- device spec (memory, compute) က အကန့်အသတ် ရှိသောကြောင့် model size ကို
  target device ပေါ်တွင် တိုင်းတာပြီးမှ ဆုံးဖြတ်ပါ

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. edge AI ၏ အားသာချက် ၃ ခု ပြောပါ။
2. federated learning တွင် device မှ ဘာကို ပို့သလဲ။
3. Jetson ပေါ်တွင် model optimize ရန် မည်သည့် tool ကို သုံးသလဲ။
4. "hospital များ — လူနာ ဒေတာ မထွက်ဘဲ model လေ့ကျင့်" — ဘာ technique လဲ။

**အဖြေ** — 1) latency, privacy, bandwidth · 2) gradient (local update) · 3) TensorRT · 4) Federated learning

## Technical review notes

- Jetson module ၏ power/memory specification များကို ဤတွင် ဂဏန်း မဖော်ပြထားပါ — product generation ပေါ် မူတည်ပါသည် (needs verification per module)။
- Federated learning privacy guarantee သည် aggregation method နှင့် threat model ပေါ် မူတည်ပါသည် — absolute privacy အဖြစ် မသတ်မှတ်ပါနှင့်။

## References

- Jetson modules: https://developer.nvidia.com/embedded/jetson-modules
- JetPack SDK: https://developer.nvidia.com/jetpack
- DeepStream: https://developer.nvidia.com/deepstream-sdk
- NVIDIA FLARE (federated learning): https://nvidia.github.io/NVFlare/
- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135/7
