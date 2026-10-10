# Module 02 — GPU Resource Management & Virtualization (Chapter 2)

GPU တစ်လုံးသည် ဈေးကြီးပြီး workload အများစုက တစ်လုံးလုံး မလိုအပ်ပါ။
တစ်ချိန်တည်းတွင် workload များ မရောထွားဘဲ လုံခြုံစွာ ခွဲဝေ အသုံးပြုရန်လည်း
လိုအပ်ပါသည်။ ဤ chapter သည် GPU ကို ခွဲဝေခြင်းနှင့် မျှဝေခြင်း နည်းလမ်းများကို
နှိုင်းယှဉ် ရွေးချယ်တတ်စေရန်နှင့် Kubernetes ပေါ်တွင် GPU workload စီမံနိုင်စေရန်
ရည်ရွယ်ပါသည်။

## ဘာပြဿနာကို ဖြေရှင်းပေးသလဲ

GPU node တစ်ခုပေါ်တွင် —
- inference service သေးသေး ၃ ခုက GPU တစ်လုံးလုံး မလိုပါ
- tenant ၂ ခု၏ workload ကို တစ်ခုနှင့်တစ်ခု မထိခိုက်စေချင်ပါ
- training job တစ်ခုက GPU အားလုံး လိုပါသည်

ဆိုလိုသည်မှာ **ခွဲဝေမှု (partitioning)**, **မျှဝေမှု (sharing)**, **သီးခြားခွဲမှု
(isolation)**, **စီစဉ်မှု (scheduling)** — ၄ မျိုးလုံး လိုအပ်ပါသည်။
နည်းလမ်းများ အားလုံးက တစ်နေရာတည်းတွင် အလုပ်မလုပ်ပါ — တစ်ခုချင်းက
hardware အဆင့် မတူပါ။

## နည်းလမ်း ၅ မျိုး — ဘယ်အဆင့်မှာ အလုပ်လုပ်သလဲ

ဤနည်းလမ်း ၅ မျိုးကို ရောထွား မမြင်ပါနှင့် — တစ်ခုချင်းက ကွဲပြားသော
software/hardware အဆင့်တွင် အလုပ်လုပ်ပါသည်။

| နည်းလမ်း | အလုပ်လုပ်သော အဆင့် | isolation | အားသာချက် | ကန့်သတ်ချက် |
|---|---|---|---|---|
| **MIG** | hardware (SM/memory ခွဲ) | အပြည့် | guaranteed memory/QoS, fault isolation | instance ပြောင်းလျှင် reset လို; profile ကြိုတင် သတ်မှတ် |
| **MPS** | CUDA runtime (process မျှဝေ) | partial | parallel execution, context ကုန်ကျ နည်း | memory ကို အပြည့် မခွဲ; one process fault က အားလုံး ထိခိုက်နိုင် |
| **Time-slicing** | driver/scheduler (အလှည့်ကျ) | မရှိ | config လွယ်, ဘယ် workload မဆို | context switch ကုန်ကျ; latency မတည်ငြိမ် |
| **vGPU** | hypervisor (VM ခွဲ) | VM အဆင့် | VM-based desktop များအတွက် | license လို; hypervisor support လို |
| **Exclusive mode** | device allocation (ရှေ့ဆုံးမှ ချိန်း) | အပြည့် | performance အပြည့်, ရိုးရှင်း | GPU တစ်ခု = job တစ်ခု (ကုန်ကျ များ) |

ရွေးချယ်ရာတွင် မေးရမည့် မေးခွန်းများ —
1. memory သီးခြား အာမခံချက် လိုသလား? (လိုပါက MIG)
2. VM-based လား? (ဟုတ်ပါက vGPU)
3. ရိုးရှင်းမှုနှင့် ကုန်ကျစရိတ် ပိုအရေးကြီးသလား? (time-slicing)

## MIG — ဘယ်လို အလုပ်လုပ်သလဲ

MIG (Multi-Instance GPU) က GPU ကို hardware level တွင် ခွဲပါသည် —
SM partition, memory slice, L2 cache, copy engine ကို instance တစ်ခုချင်းအတွက်
သီးခြား ခွဲဝေပေးသောကြောင့် instance ချင်း performance မထိခိုက်ပါ။

![without MIG](../images/mig-multiuser-before.png)
*sharing without MIG — fault က အားလုံး ထိခိုက်နိုင်*

![with MIG](../images/mig-multiuser-after.png)
*MIG instances — hardware-level isolation*

- Ampere generation နောက်ပိုင်း data center GPU များတွင် ရရှိနိုင်ပါသည်
- GPU instance profile များ (ဥပမာ `1g.10gb`, `2g.20gb`, `7g.80gb`) ကို
  ခွဲဝေစနစ်အတိုင်း instance အရေအတွက် ကျလာပါမည်
- profile စာရင်းနှင့် instance အမြင့်ဆုံး အရေအတွက်က GPU model ပေါ် မူတည်ပါသည်

## လက်တွေ့ ပြင်ဆင်မှု — MIG configuration

```bash
# 1) MIG mode ဖွင့် — node reboot လိုအပ်နိုင်ပါသည်
sudo nvidia-smi -mig 1

# 2) ဤ GPU အတွက် ရရှိနိုင်သော instance profile များ စစ်ဆေး
sudo nvidia-smi mig -lgip

# 3) instance ဖန်တီး — ဥပမာ profile ID ၂ ခု (ID များက GPU ပေါ် မူတည်!)
sudo nvidia-smi mig -cgi 9,9 -C

# 4) ဖန်တီးပြီးသော instance များ ကြည့်ရှု
nvidia-smi mig -lgi

# 5) instance ဖျက် (GI ID ဖြင့်)
sudo nvidia-smi mig -dgi -gi 0

# workload ထည့်ပြီးနောက် instance များ အဖြစ် မြင်ရ
nvidia-smi -L
```

⚠️ profile ID (ဥပမာ `9,9`) က ဥပမာသာ ဖြစ်ပါသည် — GPU generation ပေါ် မူတည်၍
ပြောင်းလဲပါသည်။ `nvidia-smi mig -lgip` ဖြင့် မိမိ GPU ၏ ID ကို အမြဲ စစ်ဆေးပါ။
ထို့အပြင် MIG mode ဖွင့်ရုံနှင့် instance မထွက်လာပါ — instance ဖန်တီးမှ
workload ထည့်နိုင်ပါမည် (enabled ≠ usable)။

## MPS နှင့် time-slicing — ဘယ်အခါ သုံးမလဲ

**MPS** (Multi-Process Service) က process များကို CUDA context တစ်ခုတည်း အောက်တွင်
တပြိုင်နက် အလုပ်လုပ်စေပါသည် — context switch မလိုတော့ဘဲ kernel များ တကယ်
overlap ဖြစ်စေပါသည်။ သို့သော် memory limit ကို process အလိုက် ခွဲမပေးနိုင်ပါ။

**Time-slicing** က GPU ကို အချိန် slot အလိုက် အလှည့်ကျ ပေးခြင်း ဖြစ်ပါသည် —
config လွယ်သော်လည်း context switch ကုန်ကျပြီး latency မတည်ငြိမ်ပါ။

```yaml
# time-slicing ဖြင့် မျှဝေမည်ဆိုပါက device plugin config ဥပမာ
# (configmap + restart — cluster version ပေါ် မူတည်)
sharing:
  timeSlicing:
    resources:
    - name: nvidia.com/gpu
      replicas: 4
```

## vGPU — VM-based environment များအတွက်

vGPU က hypervisor (NVIDIA virtual GPU platform) က GPU ကို VM များကြား
mediated ဖြင့် ခွဲဝေပေးခြင်း ဖြစ်ပါသည်။

- သုံးရန် — supported GPU, vGPU software license, compatible hypervisor version လိုအပ်ပါသည်
  (license model နှင့် version support က ပြောင်းလဲနိုင်သောကြောင့် vendor documentation မှ အတည်ပြုပါ)
- သင့်သော အသုံးပြုမှု — VDI (design/desktop များ), multi-tenant VM environment
- စီမံရန် — profile assignment, license provisioning, per-VM monitoring

MIG (bare-metal hardware partition) နှင့် vGPU (hypervisor-mediated) က မတူပါ —
စာမေးပွဲ/interview တွင် "VM-based desktop" ဆိုပါက vGPU၊ "bare-metal isolated
inference" ဆိုပါက MIG ဟု ဖြေလေ့ ရှိပါသည်။

## Kubernetes ပေါ်တွင် GPU workload စီမံခြင်း

Kubernetes က GPU ကို ကိုယ်တိုင် မသိပါ — NVIDIA Device Plugin က GPU ကို
extended resource `nvidia.com/gpu` အဖြစ် scheduler အား တင်ပြပေးပါသည်။

![GPU Operator](../images/gpu-operator-arch.png)
*ပုံ — GPU Operator architecture (ရင်းမြစ်: NVIDIA Developer Blog)*

```yaml
# pod တစ်ခုတွင် GPU ၂ လုံး တောင်းဆိုခြင်း
apiVersion: v1
kind: Pod
metadata:
  name: training-job
spec:
  containers:
  - name: trainer
    image: nvcr.io/nvidia/pytorch:xx.xx-py3
    resources:
      limits:
        nvidia.com/gpu: 2      # GPU ၂ လုံး (requests = limits ဖြစ်ရမည်)
```

```yaml
# MIG instance တစ်ခု တောင်းဆိုပါက resource name ပြောင်းပါ
resources:
  limits:
    nvidia.com/mig-1g.10gb: 1
```

လက်တွေ့တွင် GPU Operator က driver, container toolkit, device plugin,
DCGM exporter ကို တစ်ပြိုင်နက် တပ်ဆင်/စီမံပေးပါသည် — component တစ်ခုချင်း ကိုယ်တိုင်
တပ်ခြင်းထက် lifecycle စီမံရန် လွယ်ကူပါသည်။

စီမံခန့်ခွဲမှု အလေ့အထ များ —
- GPU node များကို taint/toleration သို့ nodeSelector ဖြင့် သီးခြား ထားပါ
- ResourceQuota ဖြင့် namespace/tenant အလိုက် GPU အရေအတွက် ကန့်သတ်ပါ
- DCGM exporter → Prometheus → Grafana ဖြင့် utilization, temperature, power ကို စောင့်ကြည့်ပါ
- scheduler ကို bin-packing (GPU ပြည့်အောင်) ဖြင့် cluster ကုန်ကျစရိတ် လျှော့ချနိုင်ပါသည်

## လုပ်ငန်းခွင် သတိပေးချက်များ

- **MIG profile ပြောင်းလျှင်** instance များ ဖျက်ပြီး ပြန်ဖန်တီးရပါမည် —
  production service အတွက် maintenance window ကြိုတင် ထားပါ
- **time-slicing တွင်** တစ်ခု memory ပြည့်ပါက အားလုံး ထိခိုက်နိုင်ပါသည် —
  guaranteed isolation လိုပါက မသုံးပါနှင့်
- **`nvidia.com/gpu: N`** တွင် limits = requests ဖြစ်ရပါမည် — GPU က
  extended resource ဖြစ်၍ overcommit မလုပ်နိုင်ပါ
- MPS က ဘယ် GPU architecture ပေါ်မှာ ဘယ်လောက် အလုပ်လုပ်သလဲ ဆိုတာက
  generation/driver version ပေါ် မူတည်ပါသည် — အသုံးမပြုမီ အတည်ပြုပါ

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. MIG နှင့် vGPU ၏ အဓိက ကွာခြားချက် (အလုပ်လုပ်သော အဆင့်) ကို ပြောပါ။
2. MPS က မည်သည့် အရာကို မျှဝေသလဲ။
3. `nvidia.com/gpu: 2` ဆိုခြင်းက ဘာကို ဆိုလိုသလဲ။
4. guaranteed memory isolation လိုသော multi-tenant inference အတွက် မည်သည့်နည်းလမ်း ရွေးမလဲ။

**အဖြေ** — 1) MIG = hardware partition၊ vGPU = hypervisor-mediated (license လို) · 2) CUDA context/process မျှဝေ · 3) GPU ၂ လုံး တောင်းဆိုခြင်း (limits = requests) · 4) MIG

## Technical review notes

- ဤနေရာမှ ပြသထားသော MIG profile ID နှင့် instance အမြင့်ဆုံး အရေအတွက်သည် GPU model ပေါ် မူတည်ပါသည် (needs verification per target GPU — `nvidia-smi mig -lgip` ဖြင့် စစ်ဆေးနိုင်ပါသည်)။
- vGPU licensing model နှင့် supported hypervisor versions သည် ပြောင်းလဲနိုင်ပါသည် — vendor documentation မှ အတည်ပြုပါ (မဖော်ပြထားပါ)။
- time-slicing device plugin config သည် GPU Operator/device plugin version ပေါ် မူတည်၍ ပုံစံ ကွဲပြားနိုင်ပါသည်။

## References

- MIG User Guide: https://docs.nvidia.com/datacenter/tesla/mig-user-guide/
- vGPU Software: https://www.nvidia.com/en-us/enterprise/virtual-gpu-technology/
- GPU Operator: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/
- k8s-device-plugin: https://github.com/NVIDIA/k8s-device-plugin
- MPS: https://docs.nvidia.com/deploy/mps/
