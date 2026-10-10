# Module 04 — Exercise

## E1. GPU enable sequence
K8s cluster အသစ်တွင် GPU workload ပထမဆုံး run နိုင်ရန် တပ်ဆင်ရမည့် component ၃ ခု အစဉ်အတိုင်း ရေးပါ

## E2. Tool matching
1. job queue/gang scheduling လိုအပ် → ?
2. ML pipeline အဆင့်ဆင့် K8s ပေါ် → ?
3. experiment/parameter မှတ်တမ်း + model registry → ?

## E3. Manifest + MIG
MIG profile `1g.10gb` တစ်ခု တောင်းဆိုသော pod YAML ရေးပါ

## E4. Topology (deployment model) ရွေးချယ်မှု
1. 24/7 LLM training ကြီး — data sovereignty လို → 2. startup တစ်ခု experiment/spike → 3. steady training (on-prem) + peak တွင် cloud burst →
တစ်ခုချင်း topology (On-prem/Cloud-native/Hybrid) ရွေးပြီး အကြောင်းပြပါ
