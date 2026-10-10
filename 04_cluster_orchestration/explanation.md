# Module 04 — Cluster Orchestration & Scalability (Chapter 4)

node အနည်းငယ်က SSH ဖြင့် တစ်လုံးချင်း စီမံ၍ ရပါသည် — node ၁၀ ကျော်
workload ရောထွားလာပါက cluster manager တစ်ခု မဖြစ်မနေ လိုအပ်လာပါသည်။
ဤ chapter သည် Kubernetes ဖြင့် GPU cluster စီမံခြင်း၊ Helm/operator/autoscaling
tool များနှင့် Slurm, Kubeflow, MLflow တို့၏ အခန်းကဏ္ဍ ကွာခြားချက်ကို
ရှင်းလင်းပါမည်။

## Orchestration က ဘာကို ဖြေရှင်းပေးသလဲ

- job ကို ရရှိနိုင်သော node များပေါ် ဖြန့်ဝေပေးခြင်း
- node/pod ပျက်ပါက recovery လုပ်ပေးခြင်း
- resource ကို namespace/user အလိုက် ကန့်သတ်ပေးခြင်း (quota)
- dependency/version များကို ပုံမှန်ဖြစ်အောင် ထိန်းပေးခြင်း

```
user/job → orchestrator (K8s / Slurm) → node များ → GPU ခွဲဝေ + monitor
```

## Kubernetes ပေါ်တွင် GPU support ဖွင့်ခြင်း

GPU workload ပထမဆုံး run နိုင်ရန် component ၃ ခု အစဉ်အတိုင်း လိုအပ်ပါသည် —

1. **GPU driver** (node အဆင့်) — hardware ကို အလုပ်လုပ်စေခြင်း
2. **NVIDIA Container Toolkit** — container ထဲကနေ GPU ကို မြင်ရစေခြင်း
3. **Device Plugin** — GPU ကို Kubernetes resource အဖြစ် scheduler အား တင်ပြခြင်း

၁–၃ အားလုံးကို **GPU Operator** ဖြင့် တစ်ပြိုင်နက် တပ်ဆင်/ထိန်းသိမ်းနိုင်ပါသည်။

```yaml
resources:
  limits:
    nvidia.com/gpu: 1     # GPU တစ်လုံး — requests = limits ဖြစ်ရမည်
```

MIG ဆိုပါက resource name ကို profile အလိုက် သုံးပါ —

```yaml
resources:
  limits:
    nvidia.com/mig-1g.10gb: 1
```

## Helm, Operator, Autoscaling — အခန်းကဏ္ဍ များ

| Tool | ဘာလုပ်ပေးသလဲ |
|---|---|
| **Helm** | Kubernetes app များကို package (chart) အဖြစ် တပ်ဆင်/upgrade |
| **Operator** | domain logic ပါသော controller — cluster state ကို ကိုယ်တိုင် ထိန်းပေး |
| **Cluster Autoscaler** | pending pod ရှိပါက node ထပ်၊ idle ဆိုပါက ဖယ် |

- GPU Operator သည် operator pattern ၏ ဥပမာ — install တစ်ခါ၊ lifecycle အလိုက် စီမံပေးပါသည်
- GPU node က ကုန်ကျစရိတ် မြင့်သောကြောင့် autoscaling က ကုန်ကျစရိတ် ထိန်းချုပ်မှုနှင့်
  တိုက်ရိုက် ဆက်စပ်ပါသည် — workload မရှိချိန်တွင် scale down လုပ်နိုင်ပါသည်

## Slurm, Kubeflow, MLflow — ဘာကွာသလဲ

| Tool | အခန်းကဏ္ဍ | ပုံမှန် အသုံးပြုသူ |
|---|---|---|
| **Slurm** | HPC job scheduler (queue, gang scheduling) | research/HPC cluster |
| **Kubeflow** | Kubernetes ပေါ် ML pipeline/workflow | cloud-native MLOps team |
| **MLflow** | experiment tracking, model registry | framework-agnostic မည်သည့် team |

```
Slurm    = job ကို ဘယ်အခါ ဘယ် node ပေါ် run မလဲ (scheduling)
Kubeflow = pipeline အဆင့်များကို Kubernetes ပေါ် orchestrate
MLflow   = experiment parameter/result/model version မှတ်တမ်း
```

လက်တွေ့တွင် hybrid ဖြစ်မှု များပါသည် — training က Slurm (HPC node)၊
serving က Kubernetes။ tool ရွေးချယ်မှုက team workflow ပေါ် မူတည်ပါသည်။

## Cluster topology — deployment model ၃ မျိုး

| အချက် | On-premises | Cloud-native | Hybrid |
|---|---|---|---|
| ကုန်ကျ | CapEx (ကြိုဝယ်) | OpEx (သုံးမှ ပေး) | ပေါင်းစပ် |
| Scale | နှေး (ဝယ်ယူမှု ပေါ်) | မြန် (provisioning) | အလယ် |
| Control | အပြည့် | ကန့်သတ်ချက် ရှိ | အလယ် |
| သင့်သော အသုံး | steady 24/7 load | burst, experiment, dev | peak ကို cloud ဖြင့် ဖြည့် |

- **On-premises** — data control အပြည့်၊ steady workload အတွက် TCO ပြန်ရနိုင်
- **Cloud-native** — elasticity ကောင်းသော်လည်း data egress နှင့် long-running job ကုန်ကျ စဉ်းစားရ
- **Hybrid** — steady load က on-prem၊ burst က cloud — data sync နှင့် tooling ညီညွတ်မှု စိန်ခေါ်မှု ရှိ

⚠️ deployment model (အထက်ပါ ၃ မျိုး) ကို physical topology (single-node, multi-node
scale-out, fat-tree fabric) နှင့် မရောမိပါစေနှင့် — ၂ မျိုးက မတူသော ရှုထောင့် ဖြစ်ပါသည်။

## လုပ်ငန်းခွင် သတိပေးချက်များ

- GPU က extended resource ဖြစ်သောကြောင့် `limits = requests` ဖြစ်ရပါမည်
- MIG resource name များက cluster ရဲ့ MIG strategy (single/mixed) ပေါ် မူတည်ပါသည်
- autoscaling မလုပ်မီ pod အတွက် node capacity ကို အရင် တွက်ပါ —
  GPU node တစ်ခု၏ GPU/RAM ပမာဏက SKU ပေါ် မူတည်ပါသည်

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. GPU Operator မသုံးပါက component ၃ ခု ကိုယ်တိုင် တပ်ရမည် — ဘာတွေလဲ။
2. Slurm နှင့် Kubeflow ၏ အဓိက ကွာခြားချက် ပြောပါ။
3. GPU cluster တွင် autoscaling က ဘာကြောင့် အရေးကြီးသလဲ။
4. "steady 24/7 training, data sovereignty" — deployment model ဘာရွေးမလဲ။

**အဖြေ** — 1) driver, container toolkit, device plugin · 2) job scheduler vs ML pipeline platform · 3) ကုန်ကျစရိတ် ထိန်းချုပ်ပြီး pending job မကျန်စေရန် · 4) On-premises

## Technical review notes

- MIG resource name ဥပမာ (`nvidia.com/mig-1g.10gb`) သည် GPU profile naming ပေါ် မူတည်ပါသည် — target GPU ၏ profile များနှင့် ကိုက်ညီမှု စစ်ဆေးပါ (needs verification per GPU)။
- Kubeflow/MLflow version-specific install steps များကို ဤတွင် မဖော်ပြထားပါ — official docs မှ အတည်ပြုပါ။

## References

- GPU Operator: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/
- Helm: https://helm.sh · Kubeflow: https://www.kubeflow.org · MLflow: https://mlflow.org
- Slurm: https://slurm.schedmd.com
- Cluster Autoscaler: https://github.com/kubernetes/autoscaler
- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135/4
