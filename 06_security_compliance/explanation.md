# Module 06 — Security, Compliance & Data Governance (Chapter 6)

AI platform တစ်ခုတွင် တန်ဖိုးကြီး hardware၊ လျှို့ဝှက် dataset/model နှင့်
multi-tenant အသုံးပြုသူ များ တပြိုင်နက် ရှိနေပါသည် — တစ်နေရာတည်း အားကိုးခြင်း
(security ကို တစ်ခုတည်းသော tool ဖြင့် ဖြေရှင်းခြင်း) က လုံလောက်ခြင်း မရှိပါ။
ဤ chapter သည် layered security control များ၊ workload isolation၊ data protection
နှင့် GDPR/HIPAA/FedRAMP ကဲ့သို့ compliance framework များကို ရှင်းလင်းပါမည်။

## Layered defense — ဘာကြောင့် အလွှာခွဲ ကာကွယ်ရသလဲ

```
┌────────────────────────────────────────────────────┐
│ ⑥ Policy & Compliance   RBAC, audit, GDPR/HIPAA   │
├────────────────────────────────────────────────────┤
│ ⑤ Data protection       encryption (rest/transit)  │
├────────────────────────────────────────────────────┤
│ ④ Workload isolation    namespace/VM/MIG           │
├────────────────────────────────────────────────────┤
│ ③ Supply chain          image signing, CVE scan    │
├────────────────────────────────────────────────────┤
│ ② Platform trust        secure boot, attestation   │
├────────────────────────────────────────────────────┤
│ ① Physical/Network      OOB management, segment    │
└────────────────────────────────────────────────────┘
```

တစ်ခု ကျော်ပါက နောက်တစ်ခုက ကာကွယ်ပေးခြင်း — defense in depth — က
ရည်ရွယ်ချက် ဖြစ်ပါသည်။

## Platform trust နှင့် supply chain

- **Secure Boot + TPM** က firmware မှ OS အထိ signature စစ်ပေးပါသည် —
  ပြင်ပြစ်ထားသော code မဝင်နိုင်ပါ
- **Attestation** က node သည် ပြင်ဆင်ထားသော state အတိုင်း ဖြစ်ကြောင်း
  remote အတည်ပြုပေးပါသည်
- **Image supply chain** —

```
build → sign (cosign/Notary) → CVE scan → registry policy → deploy (verify)
```

NGC catalog က signed/validated image များ ပေးပါသည် (Module 08) —
container မဝင်မီ signature နှင့် vulnerability ကို စစ်ဆေးခြင်းက production
cluster ၏ အခြေခံ အလေ့အထ ဖြစ်ပါသည်။

## Isolation — ဘယ်အဆင့်မှာ ခွဲမလဲ

| level | နည်းလမ်း | ထိရောက်မှု |
|---|---|---|
| container | namespace/cgroups | lightweight, ပြန့်ပွားမြန် |
| VM | hypervisor | ပိုခိုင်မာ |
| GPU | **MIG** (hardware partition) | GPU memory/SM သီးခြား |
| cluster | namespace + RBAC + NetworkPolicy | tenant ခွဲ |

multi-tenant GPU cluster တွင် MIG (hardware isolation) နှင့် namespace/RBAC
(policy isolation) ကို ပေါင်းသုံးခြင်းက အကောင်းဆုံး လက်တွေ့ ချဉ်းကပ်မှု ဖြစ်ပါသည်။

## Encryption နှင့် policy enforcement

| data state | နည်းလမ်း |
|---|---|
| At rest | storage/disk encryption (LUKS, S3 server-side encryption) |
| In transit | TLS/IPsec (fabric အတွင်းလည်း ထည့်စဉ်းစားပါ) |
| In use | confidential computing (TEE) — data center GPU များတွင် ရရှိနိုင် |

Kubernetes ပေါ်တွင် policy များ —

- **RBAC** — role/rolebinding ဖြင့် ဘယ်သူ ဘာလုပ်နိုင်မည် ကန့်သတ်
- **NetworkPolicy** — pod ကြား traffic ထိန်းချုပ်
- **Admission policy** (OPA/Gatekeeper ကဲ့သို့) — privileged pod မဝင်နိုင်အောင် ပိတ်ဆို့
- **ResourceQuota / LimitRange** — resource လွန်ကဲစွာ သုံးစွဲမှု ကာကွယ်

```yaml
# RBAC ဥပမာ — pod ဖတ်/ဖန်တီးခွင့်သာ ပေးခြင်း
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "get"]
```

## Compliance framework များ

| Framework | ဘာကို ကာကွယ်သလဲ | အဓိက တာဝန် |
|---|---|---|
| **GDPR** | EU ကိုယ်ရေး ဒေတာ | consent, deletion, breach notification |
| **HIPAA** | US ကျန်းမာရေး ဒေတာ | access control, audit, encryption |
| **FedRAMP** | US government cloud | authorization, continuous monitoring |

⚠️ compliance သည် tool တပ်ဆင်ခြင်း မဟုတ်ပါ — **evidence** ဖြစ်ပါသည်။
audit log, access record, encryption proof များ စုဆောင်း သိမ်းဆည်းထားနိုင်မှ
"compliant" ဟု ပြောနိုင်ပါသည်။

## လုပ်ငန်းခွင် သတိပေးချက်များ

- regulation အသေးစိတ် (article/section များ) ကို legal/official source မှသာ
  အတည်ပြုပါ — ဤနေရာမှာ ခြုံငုံ ရှင်းလင်းချက်သာ ဖြစ်ပါသည်
- image scanning tool များ၏ detection capability က tool/version ပေါ် မူတည်ပါသည်
- confidential computing feature ရရှိနိုင်မှုက GPU generation နှင့် platform
  support ပေါ် မူတည်ပါသည် (needs verification per hardware)

## ကိုယ့်ကိုယ်ကို စစ်ဆေးရန်

1. multi-tenant GPU isolation အတွက် hardware + policy အဆင့် ၂ ခု ပြောပါ။
2. container image supply chain အဆင့် ၃ ခု ကို ပြောပါ။
3. GDPR က မည်သည့် ဒေတာ အမျိုးအစားကို ကာကွယ်သလဲ။
4. "compliant" ဖြစ်ကြောင်း ပြရန် ဘာလိုအပ်သလဲ။

**အဖြေ** — 1) MIG (hardware) + namespace/RBAC (policy) · 2) sign → scan → verify · 3) EU ကိုယ်ရေး ဒေတာ · 4) audit log/access record/encryption proof ကဲ့သို့ evidence

## Technical review notes

- GDPR/HIPAA/FedRAMP တွင် ဖော်ပြထားသော တာဝန်များက အကျဉ်းချုပ်ဖြစ်ပါသည် — အပြည့်အစုံ obligations များအတွက် တရားဝင် text များကို ကိုးကားပါ (ဤတွင် အသေးစိတ် မဖော်ပြပါ)။
- Encryption feature များ (FIPS mode, TEE) ၏ availability က platform/driver version ပေါ် မူတည်ပါသည်။

## References

- Kubernetes RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Sigstore/Cosign: https://www.sigstore.dev
- NVIDIA Confidential Computing: https://www.nvidia.com/en-us/data-center/confidential-computing/
- GDPR: https://gdpr.eu · HIPAA: https://www.hhs.gov/hipaa · FedRAMP: https://www.fedramp.gov
