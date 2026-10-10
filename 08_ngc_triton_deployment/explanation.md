# Module 08 — NGC, Triton Inference Server & Deployment (Chapter 8)

trained model တစ်ခုကို service အဖြစ် ဖြန့်ချိရန် model file ကို web server ထဲ
ထည့်ရုံနှင့် မလုံလောက်ပါ — batching, scaling, monitoring, versioning များ လိုအပ်ပါသည်။
ဤ chapter သည် NGC catalog အသုံးပြုပုံ၊ Triton Inference Server ၏ component များနှင့်
production deployment pattern များကို ရှင်းလင်းပါမည်။

## NGC (NVIDIA GPU Cloud) — ဘာတွေ ရနိုင်သလဲ

```
NGC catalog
├─ Containers   (CUDA, PyTorch, NeMo, Triton — validated image များ)
├─ Models       (pretrained — LLM, vision, speech)
├─ SDKs         (TAO, NeMo, Riva)
└─ Resources    (deployment guide, Helm chart)
```

- image များက CUDA/cuDNN version ကိုက်ညီအောင် ပြင်ဆင်ပြီးသား — dependency
  ပြဿနာ လျှော့ချပေးပါသည်
- validated/tested ဖြစ်သောကြောင့် production အတွက် စတင်ရာ အမှတ် ကောင်းပါသည်

```bash
# NGC CLI ဖြင့် image ဆွဲခြင်း (account/setup က NGC docs အတိုင်း)
ngc registry image pull nvidia/tritonserver:xx.xx-py3
```

⚠️ tag/version များက အချိန်နှင့်အမျှ ပြောင်းလဲပါသည် — ပြသထားသော `xx.xx` က
ဥပမာ နေရာယူထားခြင်းသာ ဖြစ်ပြီး NGC catalog မှ လက်ရှိ tag ကို ရွေးပါ။

## Triton Inference Server — ဘယ်လို အလုပ်လုပ်သလဲ

```
        client (HTTP / gRPC)
              ↓
┌────────── Triton Inference Server ───────────┐
│  Model Repository  (model/version သိမ်းရာ)   │
│  ┌──────────┐┌──────────┐┌──────────┐        │
│  │ TensorRT ││ PyTorch  ││ ONNX RT  │ backend│
│  └──────────┘└──────────┘└──────────┘        │
│  Scheduler — dynamic batching, request queue │
│  Ensemble — model များ ချိတ်ဆက် pipeline    │
└───────────────────────────────────────────────┘
```

| Component | အခန်းကဏ္ဍ |
|---|---|
| **Model repository** | model နှင့် version များ သိမ်းဆည်းရာ (filesystem/cloud) |
| **Backend** | framework engine (TensorRT, PyTorch, ONNX, Python) |
| **Scheduler** | request များ စုပြီး dynamic batching လုပ်ပေး |
| **Ensemble** | preprocess → model → postprocess ကို pipeline အဖြစ် ချိတ် |
| **API** | HTTP/gRPC — client အမျိုးမျိုး ကနေ ခေါ်နိုင် |

**Dynamic batching** က Triton ၏ အဓိက အားသာချက် ဖြစ်ပါသည် — request သေးသေး
များကို အချိန် အနည်းငယ် စုပြီး GPU ပြည့်ပြည့် သုံးစေခြင်းဖြင့် throughput
တိုးစေပြီး per-request ကုန်ကျစရိတ် လျှော့ချပေးပါသည်။ batch size နှင့်
timeout ကို model အလိုက် ချိန်ညှိနိုင်ပါသည်။

## Deployment pattern များ

```
client → ingress → load balancer → Triton pod ×N → model storage
```

| Pattern | ဘာပြဿနာ ဖြေရှင်းသလဲ |
|---|---|
| **Load balancing** | request များကို pod များကြား ဖြန့်ဝေ — GPU အားလုံး ညီမျှ |
| **Autoscaling** | traffic ပေါ် pod အရေအတွက် ပြောင်း (HPA/KEDA + GPU metric) |
| **Redundancy** | pod အများ — တစ်ခု ကျလျှင် ကျန်က ဆက်လက် ဝန်ဆောင် |
| **Failover** | health check + rolling update — downtime မရှိ ပြောင်း |
| **Model versioning** | canary/A-B — version အသစ်ကို ဖြည်းဖြည်း ဖြန့် |

production checklist —

- health/readiness probe (Triton health endpoint)
- metrics → Prometheus (request latency, GPU utilization)
- model load/unload policy — node memory မပြည့်အောင် ထိန်း
- request timeout/retry နှင့် circuit breaker
- autoscaling metric ကို request rate သို့ GPU metric ဖြင့် တိုင်းတာ

## လုပ်ငန်းခွင် သတိပေးချက်များ

- Triton command/feature များက version ပေါ် မူတည်ပါသည် — model repository
  format နှင့် config option များကို မိမိ version ၏ docs ဖြင့် အတည်ပြုပါ
- dynamic batching က throughput ကို တိုးသော်လည်း per-request latency ကို
  အနည်းငယ် တိုးစေနိုင်ပါသည် — latency SLA ရှိပါက timeout ချိန်ညှိပါ
- NGC image များက GPU architecture အလိုက် tag ကွဲနိုင်ပါသည် — target GPU နှင့်
  ကိုက်ညီသော tag ရွေးပါ

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. NGC က ပေးသော resource အမျိုးအစား ၄ မျိုး ပြောပါ။
2. Triton ၏ dynamic batching က ဘာကို တိုးတက်စေသလဲ။
3. failover အတွက် pattern ၂ ခု ပြောပါ။
4. model repository နှင့် backend ၏ ကွာခြားချက် ပြောပါ။

**အဖြေ** — 1) containers, models, SDKs, resources · 2) GPU throughput (request စုပြီး သုံးခြင်း) · 3) redundancy + health check/rolling update · 4) repository = model သိမ်းရာ၊ backend = framework engine

## Technical review notes

- NGC image tag နှင့် Triton version-specific syntax များကို ဥပမာ အဖြစ်သာ ဖော်ပြထားပါသည် — catalog/docs မှ လက်ရှိ အချက်အလက်ကို အတည်ပြုပါ။
- Triton တွင် ဖော်ပြထားသော endpoint/health URL များက version ပေါ် မူတည်နိုင်ပါသည် (needs verification per version)။

## References

- NGC: https://ngc.nvidia.com · NGC user guide: https://docs.nvidia.com/ngc/
- Triton server: https://github.com/triton-inference-server/server
- Triton docs: https://docs.nvidia.com/deeplearning/triton-inference-server/
- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135/8
