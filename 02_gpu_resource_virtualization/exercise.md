# Module 02 — Exercise

## E1. နည်းလမ်း ရွေးချယ်မှု
scenario တစ်ခုချင်းအတွက် မည်သည့် နည်းလမ်း ရွေးမလဲ (MIG/MPS/time-slicing/vGPU/exclusive) —
1. production inference service ၄ ခု (isolation လို)
2. dev/test GPU ၈ ယောက် မျှဝေ (ဈေးသက်သာရုံ)
3. design ဝန်ထမ်း ၂၀ အတွက် virtual desktop
4. LLM training job တစ်ခု (GPU အားလုံး)

## E2. MIG command sequence
MIG mode enable → instance ဖန်တီး → ကြည့် → ဖျက် — command ၅ ကြောင်း ရေးပါ

## E3. K8s manifest
GPU ၁ လုံး တောင်းဆိုသော pod YAML အပိုင်း ရေးပါ + GPU Operator မထည့်ရင် ဘာဖြစ်မလဲ

## E4. Scenario
"GPU node ပေါ် pod ၃ ခု — တစ်ခုက memory ပြည့်ပြီး crash, ကျန် ၂ ခုလည်း လိုက်ပျက်" —
ဘယ် sharing mode ကြောင့် ဖြစ်နိုင်သလဲ၊ ဘယ်လို ကာကွယ်မလဲ
