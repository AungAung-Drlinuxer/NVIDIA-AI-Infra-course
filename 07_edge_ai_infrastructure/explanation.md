# Module 07 — Edge AI Infrastructure & Integration (Chapter 7)

> edge vs cloud၊ Jetson/Orin platform၊ federated learning vs distributed inference၊
> smart city/retail/industrial IoT ဥပမာများကို ရှင်းပြပါမည်။

---

## 7.0 Edge AI ဆိုတာ ဘာလဲ (Background)

cloud datacenter မှာ မဟုတ်ဘဲ — ကား၊ စက်ရုံ၊ ဆိုင်၊ ကင်မရာ — **data ဖြစ်ပွားရာ
နေရာ အနီးတွင်** AI လုပ်ဆောင်ခြင်းကို Edge AI ဟု ခေါ်ပါသည်။

```
cloud AI:   sensor → [network] → cloud GPU → result → [network] → device
edge AI:    sensor → [edge device GPU (Jetson)] → result   (network မလို)
```

## 7.1 Edge vs Cloud ⭐

| အချက် | Edge | Cloud |
|---|---|---|
| Latency | မီလီစက္ကန့် (local) | network ပေါ် မူတည် |
| Connectivity | offline လုပ်နိုင် | အမြဲ ချိတ်ရ |
| Compute | ကန့်သတ် (power) | အကန့်အသတ် မရှိ |
| Data privacy | ဒေတာ မထွက် | ပို့ရနိုင် |
| Management | device ရာပေါင်းများ စီမံ | စီမံရ လွယ် |

- **Edge** — latency နည်း၊ privacy ကောင်း၊ bandwidth သက်သာ
- **Cloud** — scale ကြီး၊ update လွယ်၊ model ကြီးများ run

## 7.2 NVIDIA Jetson / Orin Platform ⭐

```
┌─ Jetson Orin module ──────────────────────┐
│  GPU (Ampere) + CPU + memory + ISP       │
│  power: 7W - 60W (edge budget)           │
└──────────────────────────────────────────┘
```

| Module | သုံးရာ |
|---|---|
| Orin Nano | camera, robot (power နည်း) |
| Orin NX | smart city, retail |
| AGX Orin | industrial, autonomous |

- **JetPack SDK** — CUDA, TensorRT, DeepStream (video AI) ပါဝင်
- TensorRT ဖြင့် model optimize → edge power budget အတွင်း run

## 7.3 Federated Learning vs Distributed Inference ⭐⭐

| | Federated Learning | Distributed Inference |
|---|---|---|
| ဘာလုပ် | **model ကို** data ဆီ ပို့ လေ့ကျင့် | **data ကို** model ဆီ ပို့ ခန့်မှန်း |
| data | device ထဲမှ မထွက် | inference request ပို့ |
| ရည်ရွယ် | privacy-preserving training | load ဖြန့်ဝေ serving |

```
Federated:  edge device (local train) → gradient ပဲ ပို့ → cloud ပေါင်း → model ပြန်ဖြန့်
Dist. Inference: request → load balancer → inference node များ → response
```

⚠️ ၂ ခု မရောမိပါစေနှင့် — federated = **training ဘက်**၊ distributed inference = **serving ဘက်**။ ⭐

## 7.4 Use Cases ⭐

| နယ်ပယ် | ဥပမာ | edge ဘာကြောင့် |
|---|---|---|
| **Smart city** | traffic camera, public safety | latency + bandwidth |
| **Retail** | shelf monitoring, checkout | real-time + privacy |
| **Industrial IoT** | defect detection, predictive maintenance | machine ရပ်ချိန် လျှော့ |

ပုံမှန် architecture —
```
edge (Jetson: preprocess/inference) ←→ cloud (retrain/aggregate/manage)
```
local processing + centralized service ပေါင်းစပ်မှုက ဤ chapter ၏ အဓိက သင်ခန်းစာ ဖြစ်ပါသည်။

## 5. အခြား sources ရှင်းလင်းချက်

- **စာအုပ်** — "edge က cloud ၏ အစားထိုး မဟုတ် — **ဖြည့်စွက်သူ**" —
  workload ကို latency/privacy/bandwidth ပေါ် မူတည် ခွဲဝေရန်
- **NVIDIA Jetson docs** — Jetson ကို "datacenter ရဲ့ အငယ်ဆုံး ညီအစ်ကို" အဖြစ်
  TensorRT/CUDA တူညီမှုကြောင့် ကုဒ် ပြန်မရေးရ ဟု ဖော်ပြ

## 6. Self-Check

1. Edge AI ၏ အားသာချက် ၃ ခု?
2. Federated learning တွင် ဘာက ပို့သလဲ?
3. Jetson အတွက် inference optimize tool?

**အဖြေ** — 1) latency, privacy, bandwidth · 2) gradient (data မဟုတ်) · 3) TensorRT

## 7. References & Further Reading

- Jetson: https://developer.nvidia.com/embedded/jetson-modules
- JetPack: https://developer.nvidia.com/jetpack
- DeepStream: https://developer.nvidia.com/deepstream-sdk
- Federated Learning (NVIDIA FLARE): https://nvidia.github.io/NVFlare/
