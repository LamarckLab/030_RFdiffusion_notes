<p align="left">
  <a href="./README.md">首页</a>
</p>
<p align="right">
  <a href="./docs_EN/RFdiffusion_Functions_EN.md">English</a> | <strong>中文</strong>
</p>

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

*输入输出路径*
```bash
输入目录: /data/lmk/RFdiffusion/inputs        # 存放每次运行用的 PDB
输出目录: /data/lmk/RFdiffusion/RF_outputs    # 生成的结构 PDB
日志目录: /data/lmk/RFdiffusion/RF_logs       # Hydra 运行日志
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
  inference.num_designs=5
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
  inference.num_designs=5
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
  inference.num_designs=5
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
  inference.num_designs=5
```

> **02.4 多链 motif -- 用 `/0` 表示链断开**

用 asu.pdb 的 A 链 100-200 作为 motif，B 链 72-268 整条作为 target chain，`/0` 表示新链开始
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[5-15/A100-200/5-15/0 B72-268]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
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
  inference.num_designs=5
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
  inference.num_designs=5
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
  inference.num_designs=5
```

---

### 04 Binder Design -- 结合物设计

> **04.1 单链 target -- 不指定 hotspot**

把 monomer.pdb 的 A 链作为 target，设计一条 50-70aa 的新 binder。不给 hotspot 时 binder 落点完全随机
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 50-70]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **04.2 单链 target -- hotspot 引导**

加 `ppi.hotspot_res` 指定 binder 应该结合到 target 的哪几个残基附近 (官方推荐 3-6 个)
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 50-70]' \
  'ppi.hotspot_res=[A11,A14,A18,A61,A64,A68]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **04.3 多链 target -- hotspot 引导**

用 asu.pdb 的 ABC 三链全部作为 target，hotspot 放在 A198, B198, C198
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[A73-268/0 B72-268/0 C44-268/0 50-70]' \
  'ppi.hotspot_res=[A198,B198,C198]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **04.4 推荐降噪 -- noise_scale=0.5**

官方推荐 binder design 把噪声降到 0.5 (默认 1.0)，质量和多样性的平衡点；设 0 最保守、最确定，但多样性最低
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 50-70]' \
  'ppi.hotspot_res=[A11,A14,A18,A61,A64,A68]' \
  denoiser.noise_scale_ca=0.5 \
  denoiser.noise_scale_frame=0.5 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

---

### 05 Symmetric Oligomer -- 对称同源多聚体设计

> **05.1 Cyclic C3 -- 三聚体**

`--config-name symmetry` 切换到对称专用 config；`inference.symmetry=c3` 指定 C3 对称；contig 总长必须能被 3 整除
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=c3 \
  'contigmap.contigs=[300-300]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **05.2 Cyclic C4 -- 四聚体**

把 symmetry 换成 c4，contig 总长必须能被 4 整除
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=c4 \
  'contigmap.contigs=[400-400]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **05.3 Dihedral D2 -- 二面体 (4 链)**

D2 的对称轴是三根互相垂直、相交于分子中心的 2 次旋转轴
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=d2 \
  'contigmap.contigs=[400-400]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **05.4 Tetrahedral -- 四面体对称 (12 链)**

四面体对称的对称轴是 4 根 3 次轴（顶点 ↔ 对面中心），3 根 2 次轴（相对棱的中点 ↔ 中点）
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=tetrahedral \
  'contigmap.contigs=[360-360]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

##### [RFdiffusion官方文档](https://github.com/RosettaCommons/RFdiffusion)
