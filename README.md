# Designing NVIDIA AI Infrastructure — Study Course

> Packt စာအုပ် **"Designing NVIDIA AI Infrastructure"** (ISBN 9781808080135) ကို
> reference အဖြစ် သုံးပြီး မြန်မာဘာသာဖြင့် အသေးစိတ် ရှင်းလင်းချက်များ + labs များ ပါဝင်သော course။

## ဤ Course က ဘာလဲ

AI infrastructure ကို **compute → GPU virtualization → storage/network → orchestration**
အစဉ်အတိုင်း လေ့လာသော ပြည့်စုံ course — တစ် module ချင်းတွင် —

① နောက်ခံ အသိပညာ ② architecture (diagram + official ပုံ) ③ tools usage
④ လက်တွေ့ exercise ⑤ solution ⑥ အခြား sources ရှင်းလင်းချက် ⑦ official references

📌 NCP-AII စာမေးပွဲ (AI Infrastructure Professional) နှင့်လည်း တိုက်ရိုက် ဆက်စပ်ပါသည် —
DC-Lab-Sim-course (လက်တွေ့) နှင့် ဤ course (ဒီဇိုင်း/theory) ကို ပေါင်း၍ လေ့လာပါ။

## Chapter → Module mapping

| # | Module | စာအုပ် Chapter |
|---|---|---|
| 01 | AI Infrastructure Foundations | Ch 1 — design, GPU role, CPU/GPU/DPU, pipeline acceleration, NVIDIA ecosystem |
| 02 | GPU Resource Management & Virtualization | Ch 2 — MIG, sharing/isolation, vGPU, K8s scheduling |
| 03 | Storage, Networking & Data Pipelines | Ch 3 — storage types, high-speed network, bottlenecks, ETL |
| 04 | Cluster Orchestration & Scalability | Ch 4 — K8s GPU, Helm/operators/autoscaling, Slurm/Kubeflow/MLflow, topologies |

| 05 | Performance & Monitoring | Ch 5 — Nsight, profilers, TensorRT, tuning loop |
| 06 | Security & Compliance | Ch 6 — layered controls, RBAC, GDPR/HIPAA/FedRAMP |
| 07 | Edge AI | Ch 7 — Jetson/Orin, federated learning, IoT use cases |
| 08 | NGC & Triton Deployment | Ch 8 — NGC, Triton, load balancing/failover |
| 09 | Enterprise Workflows | Ch 9 — case studies, lifecycle, feedback loop |

စာအုပ် 9 chapters အပြည့် ဖုံးလွှမ်းပြီးပါပြီ

**ရေးသားပုံ စည်းမျဉ်းများ** — `WRITING_RULES.md` (နှင့် `.hermes.md`) တွင်
course-wide style rules သတ်မှတ်ထားပါသည် — natural Burmese + English technical terms၊
problem → concept → comparison → configuration → cautions → self-check progression၊
version-dependent claims များ flag ပြုလုပ်ခြင်း၊ chapter အဆုံးတွင် "Technical review notes" ပါဝင်ခြင်း။

## အသုံးပြုပုံ

1. module တစ်ခုချင်း — `explanation.md` → `exercise.md` → `solution.md`
2. စာအုပ် မူရင်း (Packt subscription) ရှိပါက နှိုင်းယှဉ် ဖတ်ပါ
3. exercise များကို ကိုယ်ပိုင် lab (K8s/DGX/云) တွင် လက်တွေ့ စမ်းသပ်ပါ

## References

- စာအုပ် (Packt): https://subscription.packtpub.com/book/cloud-and-networking/9781808080135
- NVIDIA docs: https://docs.nvidia.com/ (module တိုင်းတွင် link သီးခြား)
- NCP-AII exam: https://www.nvidia.com/en-us/learn/certification/ai-infrastructure-professional/
- လက်တွေ့ lab course: https://github.com/AungAung-Drlinuxer/DC-Lab-Sim-course
