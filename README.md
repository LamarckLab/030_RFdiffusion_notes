## Lamarck &nbsp; &nbsp; &nbsp; 2025-8-25
### 该文档用于部署RFdiffusion
---


*01  克隆官方的源码仓库*
```bash
git clone https://github.com/RosettaCommons/RFdiffusion.git
```

*02  下载模型权重*
```bash
cd RFdiffusion
mkdir models && cd models
wget http://files.ipd.uw.edu/pub/RFdiffusion/6f5902ac237024bdd0c176cb93063dc4/Base_ckpt.pt
wget http://files.ipd.uw.edu/pub/RFdiffusion/e29311f6f1bf1af907f9ef9f44b8328b/Complex_base_ckpt.pt
wget http://files.ipd.uw.edu/pub/RFdiffusion/60f09a193fb5e5ccdc4980417708dbab/Complex_Fold_base_ckpt.pt
wget http://files.ipd.uw.edu/pub/RFdiffusion/74f51cfb8b440f50d70878e05361d8f0/InpaintSeq_ckpt.pt
wget http://files.ipd.uw.edu/pub/RFdiffusion/76d00716416567174cdb7ca96e208296/InpaintSeq_Fold_ckpt.pt
wget http://files.ipd.uw.edu/pub/RFdiffusion/5532d2e1f3a4738decd58b19d633b3c3/ActiveSite_ckpt.pt
wget http://files.ipd.uw.edu/pub/RFdiffusion/12fc204edeae5b57713c5ad7dcb97d39/Base_epoch8_ckpt.pt

Optional:
wget http://files.ipd.uw.edu/pub/RFdiffusion/f572d396fae9206628714fb2ce00f72e/Complex_beta_ckpt.pt

# original structure prediction weights
wget http://files.ipd.uw.edu/pub/RFdiffusion/1befcb9b28e2f778f53d47f18b7597fa/RF_structure_prediction_weights.pt
```

*03  根据yml文件创建conda环境 （这里把yml文件设定的环境名称从SE3nv改成了lmk_RFdiffusion）*
```bash
conda env create -f /data/lmk/RFdiffusion/env/SE3nv.yml
conda activate lmk_RFdiffusion
```

*04  安装 RFdiffusion 的核心几何深度学习组件 —— SE3Transformer*
```bash
cd env/SE3Transformer
pip install --no-cache-dir -r requirements.txt # 安装 SE3-Transformer 需要的 Python 依赖
python setup.py install # 把 SE3-Transformer 编译并安装到当前环境
cd ../..
pip install -e . # 以可编辑模式安装RFdiffusion包，这样改动源码后无需重新安装，Python会直接引用工作区的代码，便于开发与调试。
```

### 该文档用于复现RFdiffusion文档中的全部示例
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
export CUDA_VISIBLE_DEVICES = 1 # 指定使用某块GPU
```

*01  sample 1: Design an unconditional monomer 无条件的单体设计*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py 'contigmap.contigs=[150-150]' inference.output_prefix=outputs_pdb/output inference.num_designs=3
# 设计一个 150aa 的单体蛋白骨架，不附加其他结构或功能限制
```

*02  sample 2: Motif Scaffolding 蛋白基序的支架设计（在蛋白片段两端延伸骨架）*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py inference.output_prefix=outputs_pdb/output inference.input_pdb=input.pdb 'contigmap.contigs=[10-20/A12-47/10-20]' inference.num_designs=3
# 从输入文件中截取A链的12-47片段，作为motif，在两端分别延伸10-20aa的骨架

/data/lmk/RFdiffusion/scripts/run_inference.py inference.output_prefix=outputs_pdb/output inference.input_pdb=input.pdb 'contigmap.contigs=[5-15/A10-25/30-40/0 B1-150]' inference.num_designs=3
# 若输入pdb中有两条链，把a链的部分结构作为motif，保留b链1-150位, /0表示链断开

/data/lmk/RFdiffusion/scripts/run_inference.py inference.output_prefix=outputs_pdb/output inference.input_pdb=input.pdb 'contigmap.contigs=[A73-268/0 B72-268/0 C44-71/0 1-2/C74-258/0]' 'ppi.hotspot_res=[B199]' inference.num_designs=20 denoiser.noise_scale_ca=0 denoiser.noise_scale_frame=0
# 保留AB两条链，延伸C链74位点这一端，往hotspot的位置
```

*03  sample 3: Partial diffusion 部分扩散*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py inference.output_prefix=outputs_pdb/output inference.input_pdb=input.pdb 'contigmap.contigs=[150-150]' inference.num_designs=10 diffuser.partial_T=10
# 对输入的pdb添加噪声并扩散，从而实现结构上一定程度上的扰动和“变异”
# diffuser.partial_T 代表加噪的步数，加噪越多，输出和输入差别越大，加噪越少，输出和输入越相似
```

*04  sample 4: Binder design 结合物设计*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py inference.output_prefix=outputs_pdb/output inference.input_pdb=input.pdb 'contigmap.contigs=[A1-150/0 70-100]' 'ppi.hotspot_res=[A15,A11,A8]' inference.num_designs=10 denoiser.noise_scale_ca=0 denoiser.noise_scale_frame=0
# 'contigmap.contigs=[A1-150/0 70-100]' 表示保留输入PDB中a链的1-150残基，设计一条70-100aa的binder，期望结合在'ppi.hotspot_res=[A15,A11,A8]'
```

*05  sample 5: Symmetric Motif Scaffolding 对称性蛋白基序的支架设计*
```bash

```

##### [RFdiffusion官方文档](https://github.com/RosettaCommons/RFdiffusion)
