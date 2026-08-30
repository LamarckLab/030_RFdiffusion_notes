<p align="left">
  <a href="./README_EN.md">Homepage</a>
</p>
<p align="right">
  <strong>English</strong> | <a href="../RFdiffusion_Setup.md">中文</a>
</p>

## Lamarck &nbsp; &nbsp; &nbsp; 2025-08-25
#### Deploying RFdiffusion
---


*01  Cloning the official source repository*
```bash
git clone https://github.com/RosettaCommons/RFdiffusion.git
```

*02  Downloading the model weights*
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

*03  Creating the conda environment from the yml file (the environment name in the yml was changed from SE3nv to lmk_RFdiffusion)*
```bash
conda env create -f /data/lmk/RFdiffusion/env/SE3nv.yml
conda activate lmk_RFdiffusion
```

*04  Installing SE3Transformer, RFdiffusion's core geometric deep learning component*
```bash
cd env/SE3Transformer
pip install --no-cache-dir -r requirements.txt # install the Python dependencies required by SE3-Transformer
python setup.py install # compile and install SE3-Transformer into the current environment
cd ../..
pip install -e . # install the RFdiffusion package in editable mode; source changes take effect without reinstalling, since Python reads the working tree directly, which simplifies development and debugging
```

##### [RFdiffusion official documentation](https://github.com/RosettaCommons/RFdiffusion)
