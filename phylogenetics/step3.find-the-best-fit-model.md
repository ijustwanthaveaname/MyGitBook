---
description: 过滤过后，在最终构树之前，我们还需要用模型检测软件来寻找最适的构树模型。
---

# Step3.Find the best-fit model

## **modeltest（2020 MBE 最新，替代jmodeltes和prottest）**

`modeltest -d nt/aa [-u usertree] -p cpus -i input [-o output]`

## **iqtree（易用、快速、最新、ML法构树、同时可以寻找最优构树进化模型）**

`iqtree -s input.aln -m MF -nt AUTO`

