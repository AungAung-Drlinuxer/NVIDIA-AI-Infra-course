# Module 03 — Solution

## E1
1. **Object storage** (S3 — versioning ရ, ပမာဏ ကြီး)
2. **Parallel FS** (Lustre/GPFS — concurrent read)
3. **Local NVMe** (scratch/cache)

## E2
- GPU ကြား: NVLink — node ကြား: InfiniBand/RoCE — storage: NVMe-oF/high-speed Ethernet

## E3
1. `nvidia-smi` (compute state) 2. dataloader/CPU util 3. storage throughput 4. network bandwidth
- ဖြေရှင်း: prefetch + parallel loader / local cache + GDS / DALI GPU decode

## E4
Extract (S3) → validation (schema/quality) → Transform (resize/tokenize, GPU/DALI) → version → Load (parallel FS)
- validation က Extract ပြီး၊ versioning က Load မလုပ်မီ
