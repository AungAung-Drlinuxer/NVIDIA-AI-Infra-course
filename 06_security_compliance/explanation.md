# Module 06 — Security, Compliance & Data Governance (Chapter 6)

> AI platform လုံခြုံရေး — layered controls, platform trust, isolation, image verification,
> encryption, K8s RBAC, policy enforcement နှင့် GDPR/HIPAA/FedRAMP ကို ရှင်းပြပါမည်။

---

## 6.0 ဘာကြောင့် AI infrastructure လုံခြုံရေး ထူးခြားသလဲ (Background)

AI platform တွင် — ① တန်ဖိုးကြီး hardware ② လျှို့ဝှက် dataset/model ③ multi-tenant
အသုံးပြုသူ များ — ၃ ခု ပေါင်းစပ် ပါဝင်ပါသည်။ ထို့ကြောင့် **layered defense**
(တစ်နေရာတည်း အားကိုးခြင်း မပြု) လိုအပ်ပါသည်။

```
┌────────────────────────────────────────────────────┐
│ ⑥ Policy & Compliance   RBAC, audit, GDPR/HIPAA   │
├────────────────────────────────────────────────────┤
│ ⑤ Data protection       encryption (at rest/transit)│
├────────────────────────────────────────────────────┤
│ ④ Workload isolation    container/VM/MIG/namespace │
├────────────────────────────────────────────────────┤
│ ③ Supply chain          image signing, scanning    │
├────────────────────────────────────────────────────┤
│ ② Platform trust        secure boot, TPM, attestation│
├────────────────────────────────────────────────────┤
│ ① Physical/Network      OOB, segmentation          │
└────────────────────────────────────────────────────┘
```

## 6.1 Platform Trust ⭐

- **Secure Boot + TPM** (Module 07/DC-Lab-Sim 07 နှင့် ဆက်စပ်) — firmware မှ OS အထိ
  signature စစ် — ပြင်ဆင်ထားသော code မဝင်နိုင်
- **Attestation** — node က ပြင်ပြစ် မရှိကြောင်း သက်သေပြ (remote verification)
- **Measured boot** — boot chain တစ်လျှောက်လုံး hash မှတ်တမ်း

## 6.2 Isolation ⭐

| level | နည်းလမ်း | အားသာ |
|---|---|---|
| container | namespace/cgroups | lightweight |
| VM | hypervisor | strong |
| GPU | **MIG** (hardware) | GPU memory/SM သီးခြား |
| cluster | K8s namespace + network policy | tenant ခွဲ |

multi-tenant GPU cluster တွင် — MIG (hardware isolation) + namespace (policy isolation)
ပေါင်းစပ် အသုံးပြုခြင်းက လက်တွေ့ အကောင်းဆုံး ဖြစ်ပါသည်။ ⭐

## 6.3 Supply Chain — Image Verification ⭐

```
build → sign (cosign/Notary) → scan (CVE) → registry (policy) → deploy (verify)
```
- **image signing/verification** — ဘယ်သူ တည်ဆောက်သလဲ အတည်ပြု
- **CVE scanning** — လုံခြုံရေး ချို့ယွင်းချက် စစ်ဆေး
- NGC catalog က signed/validated container များ ပေးပါသည် (Module 08)

## 6.4 Encryption ⭐

| state | နည်းလမ်း |
|---|---|
| **At rest** (သိမ်းဆည်း) | disk/storage encryption (LUKS, S3 SSE) |
| **In transit** (ပို့ဆောင်) | TLS/IPsec — fabric ပေါ်လည်း |
| **In use** (အသုံးပြု) | confidential computing (TEE) — တိုးတက်ဆဲ |

## 6.5 Kubernetes RBAC & Policy ⭐

```yaml
# role: GPU node သို့ schedule လုပ်ခွင့်သာ
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "get"]
```
- **RBAC** — role/rolebinding ဖြင့် ခွင့်ပြုချက် ကန့်သတ်
- **NetworkPolicy** — pod ကြား traffic ထိန်းချုပ်
- **Admission policy** (OPA/Gatekeeper) — privileged pod, privileged container မဝင်နိုင်အောင် ပိတ်
- **Quota/LimitRange** — resource လွန်ကဲစွာ သုံးစွဲမှု ကာကွယ်

## 6.6 Compliance — GDPR / HIPAA / FedRAMP ⭐⭐

| Regulation | ဘာအတွက် | အဓိက တာဝန် |
|---|---|---|
| **GDPR** | EU ကိုယ်ရေး ဒေတာ | consent, data deletion, breach notify |
| **HIPAA** | US ကျန်းမာရေး ဒေတာ | access control, audit, encryption |
| **FedRAMP** | US government cloud | authorization, continuous monitoring |

⚠️ compliance က **နည်းပညာ မဟုတ်** — **evidence (သက်သေ အထောက်အထား)** ဖြစ်ပါသည် —
audit log, access record, encryption proof များ စုဆောင်းထားနိုင်မှ ပြောနိုင်ပါသည်။ ⭐

## 7. အခြား sources ရှင်းလင်းချက်

- **စာအုပ်** — "security က feature စာရင်း မဟုတ် — **controls ကို layer အလိုက်
  ဆက်စပ် တပ်ဆင်ရမှု**" — defense in depth ဒဿန
- **NVIDIA docs** — confidential computing (H100 CC mode) ကို "in use" data
  ကာကွယ်ရေး အဆင့်သစ် အဖြစ် ဖော်ပြ

## 8. Self-Check

1. multi-tenant GPU isolation ၂ ဆင့် (hardware + policy)?
2. image supply chain ဆင့် ၃ ခု?
3. GDPR က ဘာကို အလေးပေးသလဲ?

**အဖြေ** — 1) MIG + K8s namespace · 2) sign → scan → verify · 3) consent/deletion/breach (ကိုယ်ရေး ဒေတာ)

## 9. References & Further Reading

- Kubernetes RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Cosign/Sigstore: https://www.sigstore.dev
- NVIDIA Confidential Computing: https://www.nvidia.com/en-us/data-center/confidential-computing/
- GDPR: https://gdpr.eu · HIPAA: https://www.hhs.gov/hipaa · FedRAMP: https://www.fedramp.gov
