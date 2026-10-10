# Module 08 — Exercise

## E1. NGC ရွေးချယ်မှု
1. LLM pretrained model လို → 2. optimize ပြီး inference engine → 3. training environment container —
NGC ထဲက ဘာတွေ ယူမလဲ

## E2. Triton architecture
model repository, backend, scheduler, ensemble — တစ်ခုချင်း အခန်းကဏ္ဍ ရေးပါ

## E3. Deployment design
"traffic မြင့် inference service (99.9% uptime)" — pattern ၄ မျိုး (load balancing/autoscaling/redundancy/failover) ဘယ်လို ထည့်မလဲ စီစဉ်ပါ

## E4. Dynamic batching
request ငယ် ၁,၀၀၀/sec ကို batch မလုပ်ဘဲ ပို့ပါက ဘာဖြစ်မလဲ — Triton က ဘယ်လို ဖြေရှင်းသလဲ
