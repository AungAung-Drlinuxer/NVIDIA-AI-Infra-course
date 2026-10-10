# Module 05 — Solution

## E1
1. **Nsight Systems** (timeline) 2. **Nsight Compute** (kernel) 3. **DCGM/nvidia-smi** (state)

## E2
1. **Compute** bottleneck (kernel optimize/TensorRT)
2. **Data/Storage** (CPU preprocess — GPU စောင့်) — prefetch/DALI/cache
3. **Network** (NCCL/fabric) — NCCL tuning, link check

## E3
1) baseline latency တိုင်း → 2) nsys profile → 3) biggest gap တစ်ခု — ဥပမာ mixed precision → 4) re-measure → 5) keep/revert
- change ၃ မျိုး: FP16, kernel fusion, batch/sequence ချိန်ဆ

## E4
- layer fusion, precision calibration (FP16/INT8), kernel auto-tuning
- **accuracy ပြန်စစ်** (quantization က accuracy ကျစေနိုင်)
