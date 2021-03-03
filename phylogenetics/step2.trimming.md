---
description: 当通过多序列比对软件进行比对过后，会得到一些含有模糊位点的比对结果，这时候我们需要用一些软件取过滤掉比对结果中的低质量区域。
---

# Step2.Trimming

## **trimal（准确、更新）**

`trimal -in input.aln -out output.aln -automated1/gappyout/strict/stricplus/nogaps/noallgaps`

* automated1:自动在gappyout和stricplus中根据序列情况选择
* gappyout:删除比对中gap最多的区域
* strict & stricplus:结合gap和相似性打分，来删除得分质量欠佳的区域
* nogaps:有gap的列全删除
* noallgaps:删除全为gap的列
* clustal. Output in CLUSTAL format.
* fasta. Output in FASTA format.
* nbrf. Output in PIR/NBRF format.
* nexus. Output in NEXUS format.
* mega. Output in MEGA format.

## **Gblocks（较老的软件，以前用的比较多）**

`Gblocks xx.fasta -t=p/d/c 分别代表蛋白、DNA、密码子`

必须是 fasta 文件在前，参数在后。若没有参数，则进入交互式界面。

