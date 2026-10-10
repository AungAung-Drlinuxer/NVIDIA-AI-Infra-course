# Module 04 — Solution

## E1
① GPU driver → ② NVIDIA Container Toolkit → ③ Device Plugin (သို့ GPU Operator ဖြင့် အားလုံး)

## E2
1. **Slurm** 2. **Kubeflow** 3. **MLflow**

## E3
```yaml
resources:
  limits:
    nvidia.com/mig-1g.10gb: 1
```

## E4
1. **On-premises** — steady load, data control (CapEx ပြန်ရ)
2. **Cloud-native** — OpEx, မြန် provisioning, burst သုံးလျှင် သက်သာ
3. **Hybrid** — steady က on-prem၊ peak က cloud — data gravity + cost ချိန်ဆ
