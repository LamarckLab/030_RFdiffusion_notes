<p align="left">
  <a href="./README_EN.md">Homepage</a>
</p>
<p align="right">
  <strong>English</strong> | <a href="../RFdiffusion_Functions.md">中文</a>
</p>

## Lamarck &nbsp; &nbsp; &nbsp; 2025-08-25
#### Command reference for running RFdiffusion on the server
---

*Environment & paths*
```bash
Env on the 236 server:  lmk_RFdiffusion
Path on the 236 server: /data/lmk/RFdiffusion/scripts/run_inference.py
Env on the 117 server:  SE3nv
Path on the 117 server: /data/RFdiffusion/scripts/run_inference.py
```

*Input & output paths*
```bash
Input dir:  /data/lmk/RFdiffusion/inputs        # PDB files used for each run
Output dir: /data/lmk/RFdiffusion/RF_outputs    # generated structure PDBs
Log dir:    /data/lmk/RFdiffusion/RF_logs       # Hydra run logs
```

*GPU selection*
```bash
export CUDA_DEVICE_ORDER=PCI_BUS_ID
export CUDA_VISIBLE_DEVICES=3
```

*Input PDB files*
- `monomer.pdb` -- single-chain monomer; chain A residues 1-150 (150aa)
- `asu.pdb` -- the asymmetric unit of the SeMV (Sesbania mosaic virus) T=3 capsid, a C3 homotrimer
  - chain A residues 73-268 (196aa)
  - chain B residues 72-268 (197aa)
  - chain C residues 44-268 (225aa)

---
### 01 Unconditional Monomer -- no structural or functional constraints

Design a 150aa monomeric protein backbone with no additional structural or functional constraints.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  'contigmap.contigs=[150-150]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

---

### 02 Motif Scaffolding -- building a scaffold around a protein motif

> **02.1 Single-chain motif -- extend the backbone at both ends**

Take chain A residues 12-47 of monomer.pdb as the motif and extend it at each end with 10-20aa of random backbone.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[10-20/A12-47/10-20]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **02.2 Single-chain motif -- fix the total length**

Add `contigmap.length` to fix the total backbone length so that the random ranges cannot alter it, producing a 70aa final backbone.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[10-20/A12-47/10-20]' \
  contigmap.length=70-70 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **02.3 Single-chain motif -- constrain the total length to a range**

`contigmap.length` also accepts a range, producing a 60-70aa final backbone.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[10-20/A12-47/10-20]' \
  contigmap.length=60-70 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **02.4 Multi-chain motif -- use `/0` for a chain break**

Use chain A residues 100-200 of asu.pdb as the motif and the whole of chain B 72-268 as the target chain. `/0` marks the start of a new chain.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[5-15/A100-200/5-15/0 B72-268]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **02.5 Multi-chain motif -- hotspot guidance**

Add `ppi.hotspot_res` to specify one or more key residues on the target; the designed region orients toward them.
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

### 03 Partial Diffusion -- partial noising and denoising

> **03.1 partial_T=10 light noising**

Noise monomer.pdb and then denoise it, producing variants close to the input structure. `diffuser.partial_T` is the number of noising steps; more steps yield more variation. The contig total length must equal the residue count of the input pdb exactly — 150aa for monomer.pdb — or the run fails.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[150-150]' \
  diffuser.partial_T=10 \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **03.2 partial_T=30 heavier perturbation**

Raising partial_T to 30, roughly 60% of T, perturbs the structure considerably more, which is useful for exploring a wider fold space. The partial_T / T ratio determines how much noise is added: the closer to 1, the further the output departs from the input.
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

### 04 Binder Design -- designing a binder against a target

> **04.1 Single-chain target -- no hotspot**

Use chain A of monomer.pdb as the target and design a new 50-70aa binder. Without a hotspot, the binder position is entirely random.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 50-70]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **04.2 Single-chain target -- hotspot guidance**

Add `ppi.hotspot_res` to specify which target residues the binder should bind near; the authors recommend 3-6.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/monomer.pdb \
  'contigmap.contigs=[A1-150/0 50-70]' \
  'ppi.hotspot_res=[A11,A14,A18,A61,A64,A68]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **04.3 Multi-chain target -- hotspot guidance**

Use all three chains A, B and C of asu.pdb as the target, with hotspots at A198, B198 and C198.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=/data/lmk/RFdiffusion/inputs/asu.pdb \
  'contigmap.contigs=[A73-268/0 B72-268/0 C44-268/0 50-70]' \
  'ppi.hotspot_res=[A198,B198,C198]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **04.4 Recommended noise reduction -- noise_scale=0.5**

The authors recommend lowering the noise to 0.5 for binder design (default 1.0), the balance point between quality and diversity. A value of 0 is the most conservative and deterministic, but the least diverse.
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

### 05 Symmetric Oligomer -- symmetric homo-oligomer design

> **05.1 Cyclic C3 -- trimer**

`--config-name symmetry` switches to the symmetry-specific config and `inference.symmetry=c3` selects C3 symmetry. The contig total length must be divisible by 3.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=c3 \
  'contigmap.contigs=[300-300]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **05.2 Cyclic C4 -- tetramer**

Switch symmetry to c4. The contig total length must be divisible by 4.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=c4 \
  'contigmap.contigs=[400-400]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **05.3 Dihedral D2 -- dihedral (4 chains)**

The symmetry axes of D2 are three mutually perpendicular 2-fold rotation axes meeting at the center of the molecule.
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=d2 \
  'contigmap.contigs=[400-400]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

> **05.4 Tetrahedral -- tetrahedral symmetry (12 chains)**

Tetrahedral symmetry has four 3-fold axes (vertex ↔ center of the opposite face) and three 2-fold axes (midpoint ↔ midpoint of opposite edges).
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py \
  --config-name symmetry \
  inference.symmetry=tetrahedral \
  'contigmap.contigs=[360-360]' \
  inference.output_prefix=/data/lmk/RFdiffusion/RF_outputs/output \
  hydra.run.dir=/data/lmk/RFdiffusion/RF_logs \
  inference.num_designs=5
```

##### [RFdiffusion official documentation](https://github.com/RosettaCommons/RFdiffusion)
