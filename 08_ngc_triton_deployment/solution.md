# Module 08 — Solution

## E1
1. **Models** (pretrained) 2. **Containers** (TensorRT/Triton) 3. **Containers** (CUDA/NeMo)

## E2
- repository = model သိမ်း, backend = framework engine, scheduler = batching/queue, ensemble = pipeline ချိတ်

## E3
- ingress + LB (ဖြန့်ဝေ) → HPA/KEDA (traffic ပေါ် scale) → pod အများ (redundancy) → health check + rolling update (failover)

## E4
- GPU underutilized, latency မြင့် (overhead) — Triton dynamic batching ဖြင့် စုပြီး GPU ပြည့် သုံး
