# Module 02 — Solution

## E1
1. **MIG** (hardware isolation + QoS)
2. **Time-slicing** (isolation မလို၊ config လွယ်)
3. **vGPU** (VDI + license)
4. **Exclusive mode** (performance အပြည့်)

## E2
`nvidia-smi -mig 1` → `nvidia-smi mig -cgi 9,9 -C` → `nvidia-smi mig -lgi` → `nvidia-smi -L` → `nvidia-smi mig -dgi -gi 0`

## E3
```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```
GPU Operator မပါလျှင် device plugin/driver ကိုယ်တိုင် တပ်ရမည် — GPU resource မမြင်ရ

## E4
- **time-slicing/shared memory** ကြောင့် ဖြစ်နိုင် (memory isolation မရှိ)
- ကာကွယ် — MIG (hardware partition) သို့ memory limits/quotas + MPS memory fraction
