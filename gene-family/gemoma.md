---
description: >-
  GeMoMa是一款能够结合tblastn/mmseqs以及转录本进行基因预测的一款软件，根据其开发者发在Nucleic Acids
  Research上的论文（doi: 10.1093/nar/gkw092）来看，相比exonerate,
  geneBlastG等同源预测软件，其的准确率更高（当然作者这么说，实际还是要自己使用。。。。）
---

# GeMoMa

下面是GeMoMa的Main workflow，基本上思路就是根据参考基因组提取coding exons作为蛋白序列然后进行tblastn/mmseqs比对到Target genome，然后RNA-seq同理，整合成一个共同的模型去预测，当然RNA-seq的数据是可选的。此软件在用法上和Genewise、Exonerate的不同在于，**你需要提供参考基因组和注释，而不是仅提供一个同源蛋白，软件会自动帮你提取,并且软件自身会调用tblastn或者mmseqs，你不需要自己先用tblastn来得到一个候选区域，再去用exonerate，genewise。**

**Main workflow:**

### 一步到胃

软件本身实际上包含了3个步骤，但是作者直接整合到了run.sh/pipeline.sh这个脚本里面，因此我们可以很方便的直接调用，这两个脚本不同点在于run.sh是用的最小参数，简而言之是个加速的版本，pipeline.sh用的是标准参数，会慢一些。

```text
# Without RNA-seq data
./run.sh <search> <target-genome> <ref-anno> <ref-genome> <out-dir>
./pipeline.sh <search> <target-genome> <ref-anno> <ref-genome> <threads> <out-dir>
# With RNA-seq data
./run.sh <search> <target-genome> <ref-anno> <ref-genome> <out-dir> <lib-type> <mapped-reads>
./pipeline.sh <search> <target-genome> <ref-anno> <ref-genome> <threads> <out-dir> <lib-type> <mapped-reads>
```

用软件自带的测试数据演示下:

```text
./pipeline.sh tblastn ./tests/gemoma/target-fragment.fasta ./tests/gemoma/ref-annotation.gff ./tests/gemoma/ref-fragment.fasta 10 ./tests
```

如果是用转录组数据,那里可以填的参数有FR\_UNSTRANDED, FR\_FIRST\_STRAND, FR\_SECOND\_STRAND，代表转录组文件的方向，UNSTRANDED就是没有方向，剩下两个就是一个定义正义链一个定义负链。

