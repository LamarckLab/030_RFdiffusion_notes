## Lamarck &nbsp; &nbsp; &nbsp; 2025-08-25
#### 该文档用于记录 server 上跑 RFdiffusion 的各种命令
---

*环境 & 路径*
```bash
236 server上的环境: lmk_RFdiffusion
236 server上的路径: /data/lmk/RFdiffusion/scripts/run_inference.py
117 server上的环境: SE3nv
117 server上的路径: /data/RFdiffusion/scripts/run_inference.py
```

*GPU选择*
```bash
export CUDA_DEVICE_ORDER=PCI_BUS_ID
export CUDA_VISIBLE_DEVICES=3
```

*输入 PDB 文件*
- `monomer.pdb` -- 单链 monomer，A 链残基号 1-150 (150aa)
- `asu.pdb` -- SeMV (Sesbania mosaic virus) T=3 衣壳的非对称单元，C3 同源三聚体
  - A 链残基号 73-268 (196aa)
  - B 链残基号 72-268 (197aa)
  - C 链残基号 44-268 (225aa)

---
### 01 Unconditional Monomer -- 无条件单体设计

设计一个 150aa 的单体蛋白骨架，不附加其他结构或功能限制
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  'contigmap.contigs=[150-150]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=3
```

---

### 02 Motif Scaffolding -- 蛋白基序的支架设计

> **02.1 单链 motif -- 两端延伸骨架**

从 monomer.pdb 取 A 链 12-47 作为 motif，两端分别延伸 10-20aa 长度的随机骨架
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[10-20/A12-47/10-20]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=3
```

> **02.2 单链 motif -- 固定总长度**

加 `contigmap.length` 把骨架总长锁死，不让随机区间影响最终长度，生成 70aa 的最终骨架
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[10-20/A12-47/10-20]' \
  contigmap.length=70-70 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=3
```

> **02.3 单链 motif -- 限制总长度区间**

`contigmap.length` 也可以写区间，生成 60-70aa 的最终骨架
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[10-20/A12-47/10-20]' \
  contigmap.length=60-70 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=3
```

> **02.4 多链 motif -- 用 `/0` 表示链断开**

用 asu.pdb 的 A 链 100-200 作为 motif，B 链 72-268 整条作为 target chain，`/0` 表示新链开始
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[5-15/A100-200/5-15/0 B72-268]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=3
```

> **02.5 多链 motif -- hotspot 引导**

加 `ppi.hotspot_res` 指定 target 上一个或多个关键残基，设计的部分会朝它去
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[5-15/A100-200/5-15/0 B72-268]' \
  'ppi.hotspot_res=[B199]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=3
```

---

### 03 Partial Diffusion -- 部分扩散

> **03.1 partial_T=10 加噪扩散**

对 monomer.pdb 加噪后再去噪，输出与输入结构相近的"变异体"。`diffuser.partial_T` 是加噪步数，步数越大变异越多（contig 总长必须严格等于输入 pdb 的残基数 (monomer.pdb 是 150aa)，否则会报错）
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[150-150]' \
  diffuser.partial_T=10 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=10
```

> **03.2 partial_T=30 加大扰动**

把 partial_T 调到 30 (约 T 的 60%)，结构扰动明显加大，可用于探索更广的折叠空间（partial_T / T 这个比值决定了"加多少噪"，比值越接近 1，输出离输入越远）
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[150-150]' \
  diffuser.partial_T=30 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=10
```

---

### 04 Binder Design -- 结合物设计

> **04.1 单链 target -- 不指定 hotspot**

把 monomer.pdb 的 A 链作为 target，设计一条 70-100aa 的新 binder。不给 hotspot 时 binder 落点完全随机
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 70-100]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=10
```

> **04.2 单链 target -- hotspot 引导**

加 `ppi.hotspot_res` 指定 binder 应该结合到 target 的哪几个残基附近 (官方推荐 3-6 个)
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 70-100]' \
  'ppi.hotspot_res=[A11,A14,A18,A61,A64,A68]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=10
```

> **04.3 多链 target -- hotspot 引导**

用 asu.pdb 的 ABC 三链全部作为 target，hotspot 放在 A198,B198,C198
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[A73-268/0 B72-268/0 C44-268/0 70-100]' \
  'ppi.hotspot_res=[A198,B198,C198]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=10
```

> **04.4 推荐降噪 -- noise_scale=0.5**

官方推荐 binder design 把噪声降到 0.5 (默认 1.0)，质量和多样性的平衡点；设 0 最保守、最确定，但多样性最低
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 70-100]' \
  'ppi.hotspot_res=[A15,A50,A100]' \
  denoiser.noise_scale_ca=0.5 \
  denoiser.noise_scale_frame=0.5 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=10
```

---

### 05 Symmetric Oligomer -- 对称同源多聚体设计

> **05.1 Cyclic C3 -- 三聚体**

`--config-name symmetry` 切换到对称专用 config；`inference.symmetry=c3` 指定 C3 对称；contig 总长必须能被 3 整除
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=c3 \
  'contigmap.contigs=[300]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=2
```

> **05.2 Cyclic C4 -- 四聚体**

把 symmetry 换成 c4，总长应能被 4 整除
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=c4 \
  'contigmap.contigs=[400]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=2
```

> **05.3 Dihedral D2 -- 二面体 (4 链)**

D2 = 2n = 4 链；总长应能被 4 整除
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=d2 \
  'contigmap.contigs=[400]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=2
```

> **05.4 Tetrahedral -- 四面体 (12 链)**

总长应能被 12 整除
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=tetrahedral \
  'contigmap.contigs=[360]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=1
```

---

### 06 Symmetric Motif Scaffolding -- 对称性蛋白基序的支架设计

> **06.1 C3 对称 motif -- 用 asu.pdb 三链同位置 motif**

asu.pdb 自身是 C3 同源三聚体；从每条链取相同位置的一段做 motif，让 RFdiffusion 在 C3 对称约束下补完两端骨架。要点：contig 必须把 3 个对称单元都写清楚，每个单元里的 motif 在空间上要对应同一个对称位置
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  inference.symmetry=c3 \
  'contigmap.contigs=[50/A100-130/50/0 50/B100-130/50/0 50/C100-130/50/0]' \
  'potentials.guiding_potentials=["type:olig_contacts,weight_intra:1,weight_inter:0.1"]' \
  potentials.olig_intra_all=True \
  potentials.olig_inter_all=True \
  potentials.guide_scale=2 \
  potentials.guide_decay=quadratic \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=2
```
> [!NOTE]
> 这条命令参考官方 `examples/design_nickel.sh` (C4 版本) 改造。如果跑不通可尝试加 `inference.ckpt_override_path=/data/lmk/RFdiffusion/models/Base_epoch8_ckpt.pt` (nickel 例子说这个权重在对称 motif 任务上表现更好)。另外 SeMV ASU 是 T=3 病毒衣壳里的局部 3 重轴单元，不一定严格几何 C3 对称，跑出来需肉眼校验

---

##### [RFdiffusion官方文档](https://github.com/RosettaCommons/RFdiffusion)
