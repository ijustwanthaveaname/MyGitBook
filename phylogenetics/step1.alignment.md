---
description: 多序列比对是系统发育的基础也是第一步，一个良好的比对软件的选择，是构树的关键。下面将介绍一些主流的比对软件。
---

# Step1.Alignment

## **Mafft**

mafft是广为使用的一款软件，综合了速度、效率、准确度各种优点，主要分为精确度高的几种方法和速度较快的几种方法，在平时使用中，更多还是会考虑精确度的问题。

### **较为精确的方法**

#### **L-INS-I**

最准确的方法。适合于 &lt;200 条序列，且序列长度 &lt;~2000 aa/nt 的比对。

`mafft --localpair --maxiterate 1000 --thread 10 input [> output]`

#### **G-INS-I**

适合于序列长度相似的多序列比对。序列条数 &lt;200, 序列长度 &lt;~2000 aa/nt 。

`mafft --globalpair --maxiterate 1000 --thread 10 input [> output]`

#### **E-INS-I**

适合序列中包含较大的非匹配区域。序列条数 &lt;200, 序列长度 &lt;~2000 aa/nt 。

`mafft --ep 0 --genafpair --maxiterate 1000 --thread 10 input [> output]`

### **节约速度的方法**

#### FFT-NS-I

减少迭代次数，最大迭代次数减为 2 。

`mafft --retree 2 --maxiterate 2 input [> output]`

#### FFT-NS-2

最大迭代次数减为 0 。

`mafft --retree 2 --maxiterate 0 input [> output]`

#### FFT-NS-1

此方法非常快速，适合 &gt;2000 条序列的多序列比对。

`mafft --retree 1 --maxiterate 0 input [> output]`

#### NW-NS-I

迭代过程中不进行 FFT aproximation

`mafft --retree 2 --maxiterate 2 --nofft input [> output]`

#### NW-NS-2

`mafft --retree 2 --maxiterate 0 --nofft input [> output]`

#### NW-NS-PARTTREE-1

3 个参数都设置为最不消耗时间的类型，适合于 ~10,000 到 ~50,000 条序列的比对。

`mafft --retree 1 --maxiterate 0 --nofft --parttree input [> output]`

## **prank** 

**高准确率的比对软件，速度慢\(http://wasabiapp.org/software/prank/\)**

**使用方法如下，很简单**

`prank -DNA/-protein/-codon -F -f=paml/fasta -d=input.fas -o=out_put.fas`

使用`-convert` 参数选项还可以转换fasta, phylipi, phylips, paml, nexus or raxml.任一格式

## **MACSE** 

**适用于密码子比对的软件，也非常准确，需要用java调用，安装很简单，只需要取官网下一个jar文件即可。**

`java -jar macse.jar -prog alignSequences -seq sequences.fasta -out_NT output_NT.fasta -out_AA output_AA.fasta`

