# Module 04 — Cluster Orchestration & Scalability (Chapter 4)

> Kubernetes ဖြင့် GPU cluster စီမံခြင်း — Helm/operator/autoscaling နှင့်
> Slurm/Kubeflow/MLflow ပေါင်းစပ်မှု၊ cluster topology ဒီဇိုင်းကို ရှင်းပြပါမည်။

---

## 4.0 Orchestration ဘာကြောင့် လိုသလဲ (Background)

node ၁၀ ကျော်လာလျှင် SSH ဖြင့် တစ်လုံးချင်း စီမံ၍ မရ —
**cluster manager** က job ဖြန့်ဝေ၊ failure ကိုင်တွယ်၊ resource ခွဲဝေ ပေးရပါသည်။

```
user/job → orchestrator (K8s/Slurm) → node ရာပေါင်းများ → GPU ခွဲဝေ/monitor
```

## 4.1 Kubernetes for GPU Workloads ⭐

### Enabling GPU support
```
① NVIDIA driver (node)
② NVIDIA Container Toolkit (container က GPU မြင်ရန်)
③ Device Plugin (K8s က GPU resource အဖြစ် မြင်ရန်)
   — သို့မဟုတ် GPU Operator ဖြင့် ①②③ အားလုံး တစ်ပြိုင်နက်
```

### Requesting GPUs ⭐
```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```
- MIG ဆိုလျှင် `nvidia.com/mig-1g.10gb: 1` ကဲ့သို့ resource name
- `limits` = `requests` ဖြစ်ရမည် (GPU က extended resource)

## 4.2 Helm, Operators & Autoscaling ⭐

| Tool | ဘာလုပ် |
|---|---|
| **Helm** | K8s app package manager (chart) — GPU stack ကို တစ်ခါတည်း တပ် |
| **Operator** | domain logic + K8s controller (state ကို ကိုယ်တိုင် ထိန်း) |
| **Autoscaler** | workload ပေါ် မူတည် node အလိုအလျောက် တိုး/လျှော့ |

- **GPU Operator** = operator ဥပမာ — driver/toolkit/plugin/DCGM အားလုံး စီမံ
- **Cluster Autoscaler** — pending pod ရှိလျှင် node အသစ် ထပ်; idle ဆိုလျှင် ဖယ်
- **KEDA / MIG-aware autoscaling** — workload metric ပေါ် scale

⚠️ GPU node က ဈေးကြီး — autoscaling က **cost control** နှင့် တိုက်ရိုက် ဆက်စပ်ပါသည်။ ⭐

## 4.3 Slurm, Kubeflow, MLflow — ဘာကွာသလဲ ⭐⭐

| Tool | အခန်းကဏ္ဍ | ဘယ်မှာ သုံး |
|---|---|---|
| **Slurm** | HPC job scheduler (queue, gang scheduling) | research/HPC cluster |
| **Kubeflow** | K8s ပေါ် ML pipeline/workflow | cloud-native MLOps |
| **MLflow** | experiment tracking, model registry | ဘယ်နေရာမဆို (tool-agnostic) |

```
Slurm    = ဘယ်အခါ ဘယ် node ပေါ် run မလဲ (scheduling)
Kubeflow = pipeline အဆင့်ဆင့် ကို K8s ပေါ် orchestrate
MLflow   = experiment/parameter/result/model version မှတ်တမ်း
```

hybrid (Slurm + K8s) — training က Slurm (HPC), serving က K8s — လက်တွေ့ ဖြစ်ရပ် များပါသည်။

## 4.4 Cluster Topologies — On-premises / Cloud-native / Hybrid ⭐⭐

စာအုပ်က cluster topology ကို ၃ မျိုး ခွဲ ဖော်ပြပါသည် —

### ① On-premises (ကိုယ်ပိုင် datacenter)
```
┌─ own datacenter ─────────────────────────────┐
│  rack · IB fabric · parallel FS · Slurm/K8s  │
└──────────────────────────────────────────────┘
```
- **အားသာ** — data control အပြည့်၊ steady workload အတွက် TCO ပြန်ရ၊ latency တည်ငြိမ်
- **အားနည်း** — CapEx ကြီး၊ capacity ကြိုတင် ဝယ်ရ၊ scaling နှေး
- သင့် — 24/7 training ကြီးများ၊ data sovereignty လိုအပ်

### ② Cloud-native (public cloud)
```
┌─ cloud (NVIDIA DGX Cloud / GPU instance) ────┐
│  managed K8s · object storage · autoscaling  │
└──────────────────────────────────────────────┘
```
- **အားသာ** — OpEx (သုံးမှ ပေး)၊ မိနစ်ပိုင်း provisioning၊ elasticity
- **အားနည်း** — data egress ကုန်ကျစရိတ်၊ quota/availability ကန့်သတ်၊ long-running job ဈေးကြီး
- သင့် — burst/spike, experiment, ရှားရှားပါးပါး အသုံးပြုမှု

### ③ Hybrid (ပေါင်းစပ်) ⭐
```
on-prem (steady training)  ←→  cloud (burst/serving)
        shared data plane (replication/sync)
```
- steady load က on-prem၊ peak/burst က cloud သို့ ဖြန့်
- data gravity — dataset ကြီး ရှိရာတွင် train; inference က user နီးရာ
- **စိန်ခေါ်မှု** — data sync, network cost, tooling ညီညွတ်မှု

### topology ရွေးချယ်မှု ဇယား ⭐

| အချက် | On-prem | Cloud | Hybrid |
|---|---|---|---|
| ကုန်ကျ | CapEx (ကြို) | OpEx (သုံးမှ) | ပေါင်း |
| Scale | နှေး | မြန် | အလယ် |
| Control | အပြည့် | ကန့်သတ် | အလယ် |
| သင့် | steady, large | burst, dev | ပြောင်းလွယ် လိုအပ် |

နောက်ထပ် ဒီဇိုင်း ရှုထောင့် — single-node (POC) / multi-node scale-out (fat-tree) များက
ရုပ်ပိုင်း topology (Module 03 / DC-Lab-Sim-course Module 12) ဖြစ်ပြီး
ဤ ၃ မျိုးက **deployment model** ဖြစ်ပါသည် — ၂ မျိုး မရောမိပါစေနှင့်။ ⭐

## 5. အခြား sources ရှင်းလင်းချက်

- **စာအုပ်** — "orchestration tool ရွေးချယ်မှုက **team workflow ပေါ် မူတည်**" —
  HPC ရိုးရာ team = Slurm၊ cloud-native = K8s+Kubeflow — နည်းပညာ အလှ မဟုတ်
- **NVIDIA GPU Operator docs** — operator pattern ကို "install once, manage lifecycle"
  ဒဿန ဖြင့် ဖော်ပြ

## 6. Self-Check

1. GPU Operator မပါလျှင် ဘာတွွွေ ကိုယ်တိုင် တပ်ရမလဲ?
2. Slurm နှင့် Kubeflow ကွာခြားချက်?
3. Autoscaling က GPU cluster တွင် ဘာကြောင့် အရေးကြီးသလဲ?

**အဖြေ** — 1) driver + container toolkit + device plugin · 2) HPC scheduler vs ML pipeline platform · 3) cost control + pending job မကျန်

## 7. References & Further Reading

- GPU Operator: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/
- Helm: https://helm.sh · Kubeflow: https://www.kubeflow.org · MLflow: https://mlflow.org
- Slurm: https://slurm.schedmd.com
- Cluster Autoscaler: https://github.com/kubernetes/autoscaler
- စာအုပ်: https://subscription.packtpub.com/book/cloud-and-networking/9781808080135/4
