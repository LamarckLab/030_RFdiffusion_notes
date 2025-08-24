## Lamarck &nbsp; &nbsp; &nbsp; 2025-8-25
#### 该文档用于部署RFdiffusion
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

*03  根据yml文件创建conda环境 （这里把yml文件设定的环境名称从SE3nv改成了lmk_SE3nv）*
```bash
conda env create -f env/SE3nv.yml
conda activate lmk_SE3nv
```

*04  重建祖先序列特征*
```bash
augur traits \
--tree results/tree.nwk \
--metadata data/metadata.tsv \
--output results/traits.json \
--columns country \
--confidence
```

*05  重建祖先序列*
```bash
augur ancestral \
--tree results/tree.nwk \
--alignment results/aligned.fasta \
--output-node-data results/nt_muts.json \
--inference joint
```

*06  确定氨基酸突变*
```bash
augur translate \
--tree results/tree.nwk \
--ancestral-sequences results/nt_muts.json \
--reference-sequence config/Rabies_virus_reference.gb \
--output results/aa_muts.json
```

*07  导出汇总*
```bash
augur export v2 \
--tree results/tree.nwk \
--metadata data/metadata.tsv \
--node-data results/branch_lenths.json \
results/traits.json \
results/nt_muts.json \
results/aa_muts.json \
--colors config/colors.tsv \
--lat-longs config/lat_longs.tsv \
--auspice-config config/auspice_config.json \
--output auspice/rabies.json
```

*把生成的汇总json文件放在results文件夹中*

*08  结果可视化*
```bash
conda activate auspice
auspice build
auspice view --datasetDir <directory of results>
auspice view --datasetDir /mnt/f/1022/zika-tutorial/auspice_results/
```


##### [官方手册](https://github.com/RosettaCommons/RFdiffusion)









