## Lamarck &nbsp; &nbsp; &nbsp; 2025-8-25
#### 该文档用于实现RFdiffusion文档中的全部示例
---

*环境 & 路径*
```bash
236 server上的环境: lmk_SE3nv
236 server上的路径: /data/lmk/RFdiffusion/scripts/run_inference.py
117 server上的环境: SE3nv
117 server上的路径: /data/RFdiffusion/scripts/run_inference.py
```

*GPU选择*
```bash
export CUDA_VISIBLE_DEVICES = 1 # 在跑命令前指定使用某块GPU
```

*01  sample 1: design an unconditional monomer 无条件的单体设计*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py 'contigmap.contigs=[150-150]' inference.output_prefix=outputs_pdb/output inference.num_designs=10
# 设计一个 150aa 的单体蛋白骨架，不附加其他结构或功能限制
```

*02  sample 2: Motif Scaffolding*
```bash
/data/lmk/RFdiffusion/scripts/run_inference.py inference.output_prefix=outputs_pdb/output inference.input_pdb=input.pdb 'contigmap.contigs=[10-20/A115-147/10-20]' inference.num_designs=3
# 
```

*03  *
```bash

```

*04  *
```bash

```

##### [RFdiffusion官方文档](https://github.com/RosettaCommons/RFdiffusion)



























