# Use blast, mmseqs, diamond and hmmer to search Homologous proteins

## 本文章将介绍

**①类blast软件blast、mmseqs、diamond、hmmer的比较**

**②blast、mmseqs、diamond、hmmer的使用**

**③用四种软件的实践操作，python/R绘制韦恩图，取交集序列。**

**注:所有linux命令均在zsh下运行，而不是过时的bash**

## 正文

基因的同源鉴定有几种不同的流程方法:

* **tblastn筛选候选区域然后两端延伸+genewise/exonerate基因建模。**这种方法适合大多数的情况，好的文献当中用的都比较多，当然这本身就是基因组同源注释的一个流程之一，经常出现在新物种基因组测序然后紧接着针对某个基因家族进行分析的流程之中。
* **blastp/mmseqs/diamond/hmmer等类blastp软件（hmmer除外）的方法进行蛋白比蛋白的搜索，**得到相应的结果，我们可以采用多种软件，取交集来选择更加可靠的结果，这种方法适合有着比较良好注释的CDS的情况。

  关于tblastn+exonerate的流程，可以参考我的github：[https://github.com/ijustwanthaveaname/Genefamily\_pipeline](https://github.com/ijustwanthaveaname/Genefamily_pipeline)

接下来主要介绍类blastp的方法。

### 类blast软件的介绍、比较及使用

不多废话，直接放今年的文章

 ![fig9.png](https://upload-images.jianshu.io/upload_images/20808655-09636a9d37be3ec5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作者发现，lastal产生结果所需的时间最少。然而，当比较由进化上遥远的基因组编码的蛋白质时，它产生的结果比任何其他程序都少。产生最相似数量的RBH的是blastp和diamond以ultra-sensitive选项运行产生的结果。然而，这个选项是diamond最慢的，而very-sensityve选项提供了速度和RBH结果之间的最佳平衡。在处理真核生物基因组时，程序的提速更为明显，因为真核生物基因组编码的蛋白质数量更多。例如，Lastal在处理细菌蛋白质组时花费了约1.5%的blastp时间，在处理真核生物蛋白质组时花费了0.6%的时间，而diamond与very-sensitive的选项分别花费了7.4%和5.2%的时间。虽然用所有程序得到的RBH的估计错误率非常相似，但用MMseqs2得到的RBH在测试的程序中错误率最低，而Lastal是最快的软件。

简单来说就是：

* **Lastal**最快
* **MMseqs2**错误率最低
* **diamond**的very-sensitive兼备速度和准确率，作者认为是最优选择
* 四种软件错误率差别并不太大

**下面介绍Lastal、blastp、mmseqs2、diamond以及hmmer的用法（lastal用的比较少，不过多介绍）,用的数据来自上一篇笔记所获得，可以自行查看。**

#### **diamond用法**

```text
# 如果没有，可以通过conda或者从github安装,尽量安装最新版，旧版没有对ultra/very-sensitive选项的支持
conda install -c bioconda diamond=2.0.6
# 建库
diamond makedb --in GCF_000001635.27_GRCm39_translated_cds.faa --db mouse
# diamond blastp，这里我们还是用最准的--ultra-sensitive
diamond blastp --db mouse -q globin.faa --outfmt 6 --ultra-sensitive --threads 20 -e 1e-5 --out diamond_results.txt
```

#### **blastp用法**

```text
# 如果没有，可以通过conda或者github/ncbi官网获取。
conda install -c bioconda blast
# 建库
makeblastdb -dbtype prot -in GCF_000001635.27_GRCm39_translated_cds.faa -out mouse 
# blastp搜索
blastp -db mouse -num_threads 20 -query globin.faa -evalue 1e-5 -outfmt 6 -out blastp_results.txt
```

#### **mmseqs2用法**

```text
# 如果没有，可以通过conda/github下载最新版本
conda install -c bioconda mmseqs2
# 如果想简单一步搞定，用easy-search即可
mmseqs easy-search globin.faa GCF_000001635.27_GRCm39_translated_cds.faa mmseqsez_results.txt tmp --threads 20
# 正常流程，先建库
mmseqs createdb globin.faa queryDB
mmseqs createdb GCF_000001635.27_GRCm39_translated_cds.faa targetDB
# 建立索引，加快读取速度
mmseqs createindex targetDB tmp -s 7 --threads 10
# 搜索
mmseqs search queryDB targetDB resultDB tmp -s 7 --threads 10
# 转换成blastm8格式
mmseqs convertalis queryDB targetDB resultDB mmseqs_results.txt
```

注：mmseqs的-s选项用来指定灵敏度，7是最高,觉得速度慢，可以用 `mmseqs search qDB tDB rDB tmp --start-sens 1 --sens-steps 3 -s 7` `--start-sens`指定起始灵敏度,`--sens-steps`指定达到灵敏度`-s 7`的迭代步数，这里就是三次迭代达到最高灵敏度。 由于mmseqs没有evalue选项，因此要自己过滤阈值。

```text
# 筛选然后排序后输出到文件
awk '$11<1e-5{print}' mmseqs_results.txt |sort -g -k 11 > mmseqs_gt1e-5.txt
```

我们wc -l 看下

 ![fig10.png](https://upload-images.jianshu.io/upload_images/20808655-265b6d10dbf07484.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结果差不多，我们去下重复hits再看下。

 ![fig11.png](https://upload-images.jianshu.io/upload_images/20808655-456cbc295c03b13d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

少了很多，但是结果还是这么多的原因是在于下的是cds序列，包含了同一基因的不同转录本，所以比对到了很多冗余的结果，如果你之前选择的是最长转录本对应cds，结果会少很多。

#### **hmmer用法**

**种子序列下载** hmmer需要种子文件，可以自己序列比对生成，然后转成sto格式，或者直接取pfam官网搜索下载：[http://pfam.xfam.org/](http://pfam.xfam.org/)

 ![fig14.png](https://upload-images.jianshu.io/upload_images/20808655-c010abc8002fe9cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

输入globin，然后点go 

![fig15.png](https://upload-images.jianshu.io/upload_images/20808655-d02256a3efddf6c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击侧边栏的Alignments，如图所示，Format那里选择Stockholm即sto格式，然后点Generate即可得到种子序列。

以下流程按照自己比对生成sto格式文件来演示，其实也就是少了比对和转格式这两步

**普通流程**

```text
# 如果没有同样可以conda下载
conda install -c bioconda hmmer
# 用mafft序列比对，没有仍然可以conda下载
conda install -c bioconda mafft
mafft --localpair --maxiterate 1000 --thread 10 globin.faa > globin.aln
# 用trimal过滤一下最好，没有conda下一下
conda install -c bioconda trimal
trimal -in globin.aln -out globin_trimed.aln -automated1
# 然后用python的biopython包转fasta格式为sto格式
python -c 'from Bio import SeqIO;SeqIO.convert("./globin_trimed.aln", "fasta", "./globin_trimed.sto", "stockholm")'
# 建立谱hmm,需要提供sto格式序列比对文件
hmmbuild globin.hmm globin_trimed.sto
# 搜索cds序列
hmmsearch --cpu 10 globin.hmm GCF_000001635.27_GRCm39_translated_cds.faa  > hmm_results.txt
```

报了个错

 ![fig16.png](https://upload-images.jianshu.io/upload_images/20808655-1bd55180e59215ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sed查看一下

 ![fig17.png](https://upload-images.jianshu.io/upload_images/20808655-d60e31177260fd73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

貌似”-“不能被hmmer识别，那么我们干脆直接把”-“改成”X“就好了

```text
# 替换-为X
sed -i '/^>/!s/-/X/g' GCF_000001635.27_GRCm39_translated_cds.faa
# 继续重搜一下，成功并且没有报错
hmmsearch --cpu 10 globin.hmm GCF_000001635.27_GRCm39_translated_cds.faa  > hmm_results.txt
```

用vim看一下hmmer结果，发现结果比较少

 ![fig18.png](https://upload-images.jianshu.io/upload_images/20808655-b3614fd7df1777a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用下pfam的种子文件再试试

```text
hmmbuild globin_pfam.hmm PF00042_seed.txt
hmmsearch --cpu 10 globin_pfam.hmm GCF_000001635.27_GRCm39_translated_cds.faa  > hmm_pfam_results.txt
```

查看一下

 ![fig19.png](https://upload-images.jianshu.io/upload_images/20808655-f4fb969001b68bcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

确实也不是比对问题，hmmer得到的结果就是比较少，两次结果一模一样，如果你看不出来，提取id然后uniq一下。我们同样以1e-5为阈值，提取基因名，查看下交集，这里我们直接按照行号来用sed提取就行了。

```text
# 提取用自己下载序列比对的目标id结果
sed -n '16,33p' hmm_results.txt | awk '{print $9}' > hmm_results.id
# 提取用pfam种子得到的目标id结果
sed -n '16,33p' hmm_pfam_results.txt | awk '{print $9}' > hmm_pfam_results.id
sort -u hmm_results.id
sort -u hmm_pfam_results.id
cat hmm_results.id hmm_pfam_results.id | sort | uniq  -d
## 上面uniq命令得到的结果是完全相同的，一共18条，你也可以用wc -l看看数目是不是一样
```

接下来我们合并下不同方法结果的交集

```text
for i (blastp_results.txt diamond_results.txt mmseqs_gt1e-5.txt) {
    gawk '{print $2}' $i > ${i%.txt}.id
}
```

**ls \*id**一下，可以发现我们得到了不同方法对应的结果id

 ![fig20.png](https://upload-images.jianshu.io/upload_images/20808655-1b5541144695b2b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意下其中mmseqs的id去掉了lcl\|，因此我们可以把lcl\|加上到mmseqs的结果里的，可以**less mmseqs\_gt1e-5.id**查看下

 ![fig21.png](https://upload-images.jianshu.io/upload_images/20808655-618087c8598a14a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

加上lcl\|方便去重

```text
sed -i 's/^/lcl|/' mmseqs_gt1e-5.id
```

接下来我们用R可视化，绘制成韦恩图，查看下4种方法的重合情况。

### R/Python绘制韦恩图

个人喜欢用Python做各种分析，无论是统计还是画图。。。除非要做转录组啊什么组的要用专门的R包。不过考虑到更多做生信的国内还是喜欢用R，所以两种都试试吧。 首先是python，画Venn图的包有**matplotlib\_venn**（2-3组）和**pyvenn**（2-6组）数据. 包下载

```text
pip install matplotlib_venn
git clone https://github.com/tctianchi/pyvenn.git
```

打开python画图

```python
from pyvenn import venn
from matplotlib_venn import venn3
from matplotlib import pyplot as plt
from Bio import SeqIO


def get_id(id_path):
    id_list = []
    with open(id_path, "r") as f:
        for id in f:
            id_list.append(id.strip())
    return id_list

blast = get_id("blastp_results.id")
diamond = get_id("diamond_results.id")
mmseqs = get_id("mmseqs_gt1e-5.id")
hmmer = get_id("hmm_results.id")

# 先画除了hmmer的3个结果韦恩图
venn3(subsets = [set(blast), set(diamond), set(mmseqs)], set_labels = ("blast", "diamond", "mmseqs"), set_colors = ("b", "r", "y"))
plt.savefig("venn3_plot.png")
plt.close()
```

如图所示

 ![venn3\_plot.png](https://upload-images.jianshu.io/upload_images/20808655-a4b2f068e5501cf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

出现这种状况的原因是因为blast的结果和mmseqs的去重结果完全相同，一共有61个重叠。

我们再用pyvenn画4个的交集（其实是3个，因为blast=mmseqs）

```python
# 代码接上面
labels = venn.get_labels([set(blast), set(diamond), set(mmseqs), set(hmmer)], fill=['number'])
venn.venn4(labels, names=['blast', 'diamond', 'mmseqs', 'hmmer'])
plt.savefig("venn4_plot.png")
```

图如下：

 ![venn4\_plot.png](https://upload-images.jianshu.io/upload_images/20808655-1354f94d5d1ae61f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以后还是用pyvenn画图好了，好看很多。。。 可以用集合操作把18个重叠的基因找出来

```python
final_id = set(blast) & set(diamond) & set(mmseqs) & set(hmmer)
with open("final_id.txt", "w") as f:
    f.write("\n".join(final_id))
recs = SeqIO.parse(r"./GCF_000001635.27_GRCm39_translated_cds.faa", "fasta")
out=[rec for rec in recs if rec.id in final_id]
SeqIO.write(out, r"./predicted_mouse_globin.cds", "fasta")
```

这样我们就得到了4个软件交集的CDS序列。 （R的代码之后补上。。。有点懒了）

