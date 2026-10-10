# Module 08 — NGC, Triton Inference Server & Deployment (Chapter 8)

> NGC catalog ထဲက ဘာတွေ ယူနိုင်သလဲ၊ Triton ဘယ်လို အလုပ်လုပ်သလဲ၊
> production deployment pattern များ (load balancing, autoscaling, failover) ကို ရှင်းပြပါမည်။

---

## 8.0 Model ကနေ Service သို့ — ဘာကြောင့် serving stack လိုသလဲ

trained model တစ်ခုက **API service** ဖြစ်လာရန် — batching, scaling, monitoring,
versioning လိုအပ်ပါသည်။ model file ကို web server ထဲ ထည့်ရုံနှင့် မလုံလောက်ပါ။

## 8.1 NGC (NVIDIA GPU Cloud) ⭐

```
NGC catalog
├─ Containers   (CUDA, NeMo, RAPIDS, Triton — optimized/validated)
├─ Models       (pretrained — LLM, vision, speech)
├─ SDKs         (TAO, NeMo, Riva...)
└─ Resources    (deployment guides, Helm charts)
```

- **ပြင်ဆင်ပြီးသား** — CUDA/cuDNN version ကိုက် — dependency hell ကင်း
- **validated** — tested/performance-tuned
- `ngc` CLI ဖြင့် ဆွဲ — `ngc registry image pull nvidia/tritonserver`

## 8.2 Triton Inference Server ⭐⭐

```
        client (HTTP/gRPC)
              ↓
┌────────── Triton Inference Server ──────────┐
│  Model Repository   (model များ သိမ်းရာ)   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐        │
│  │TensorRT │ │PyTorch  │ │ONNX RT  │ backend│
│  └─────────┘ └─────────┘ └─────────┘        │
│  Scheduler (dynamic batching, model queue)   │
│  Ensemble (model များ ချိတ်ဆက် pipeline)    │
└──────────────────────────────────────────────┘
```

### Component အဓိက များ ⭐
| Component | ဘာလုပ် |
|---|---|
| **Model repository** | model version များ သိမ်း (filesystem/cloud) |
| **Backend** | framework engine (TensorRT, PyTorch, ONNX, Python) |
| **Scheduler** | request များ စုပြီး **dynamic batching** လုပ် (throughput တက်) |
| **Ensemble** | preprocess→model→postprocess ကို pipeline အဖြစ် ချိတ် |
| **API** | HTTP/gRPC (client အမျိုးမျိုး) |

⚠️ **dynamic batching** က Triton ၏ အဓိက အားသာချက် — request ငယ်များကို
စုပြီး GPU ပြည့်ပြည့် သုံးခြင်း — inference cost လျှော့ချပါသည်။ ⭐

## 8.3 Deployment Patterns ⭐⭐

```
                    ┌─ load balancer ─┐
client → ingress → │  Triton pod ×N  │ → model storage
                    └─────────────────┘
```

| Pattern | ဘာဖြေရှင်း |
|---|---|
| **Load balancing** | request ဖြန့်ဝေ — GPU အားလုံး ညီမျှ |
| **Autoscaling** | traffic ပေါ် pod တိုး/လျှော့ (KEDA/HPA + GPU metric) |
| **Redundancy** | pod အများ — တစ်ခု ကျလျှင် ကျန် ဆက် |
| **Failover** | health check + rolling update — downtime မရှိ |
| **Model versioning** | canary/A-B — version အသစ် ဖြည်းဖြည်း ဖြန့် |

## 8.4 Production checklist ⭐

- health/readiness probe (Triton `/v2/health`)
- metrics endpoint → Prometheus (request latency, GPU util)
- model load/unload policy (RAM မပြည့်အောင်)
- request timeout, retry, circuit breaker
- autoscaling metric: GPU util သို့ request rate

## 5. အခြား sources ရှင်းလင်းချက်

- **စာအုပ်** — "serving က model ထက် **service reliability** ပိုအရေးကြီး" —
  SLA/latency target သတ်မှတ်ပြီးမှ design လုပ်ရန်
- **NVIDIA Triton docs** — dynamic batching + ensemble ကို
  "throughput vs latency trade-off" ဖြင့် ချိန်ဆ အသုံးပြုရန် အကြံပြု

## 6. Self-Check

1. NGC က ဘာတွေ ပေးသလဲ (၄ မျိုး)?
2. Triton ၏ dynamic batching က ဘာကို တိုးတက်စေသလဲ?
3. failover အတွက် pattern ၂ ခု?

**အဖြေ** — 1) containers/models/SDKs/resources · 2) GPU throughput (request စု) · 3) redundancy + health-check/rolling update

## 7. References & Further Reading

- NGC: https://ngc.nvidia.com · NGC docs: https://docs.nvidia.com/ngc/
- Triton: https://github.com/triton-inference-server/server
- Triton docs: https://docs.nvidia.com/deeplearning/triton-inference-server/
- TAO Toolkit: https://developer.nvidia.com/tao-toolkit
