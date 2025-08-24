## Lamarck &nbsp; &nbsp; &nbsp; 2025-8-25
#### 该文档用于实现RFdiffusion文档中的全部示例
---


*01  sample 1: 无条件的单体设计*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py 'contigmap.contigs=[150-150]' inference.output_prefix=test_outputs/test inference.num_designs=3
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

*03  根据yml文件创建conda环境 （这里把yml文件设定的环境名称从SE3nv改成了lmk_SE3nv）*
```bash
conda env create -f env/SE3nv.yml
conda activate lmk_SE3nv
```

*04  安装 RFdiffusion 的核心几何深度学习组件 —— SE3Transformer*
```bash
cd env/SE3Transformer
pip install --no-cache-dir -r requirements.txt # 安装 SE3-Transformer 需要的 Python 依赖
python setup.py install # 把 SE3-Transformer 编译并安装到当前环境
cd ../..
pip install -e . # 以可编辑模式安装RFdiffusion包，这样改动源码后无需重新安装，Python会直接引用工作区的代码，便于开发与调试。
```

##### [RFdiffusion官方文档](https://github.com/RosettaCommons/RFdiffusion)















