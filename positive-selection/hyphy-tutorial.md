---
description: 依据官方版本2.2.0所写
---

# Hyphy tutorial

近几年来，Hyphy的使用人数越来越来多，虽然不及paml，但这款软件的一些优秀特性使得它值得受到使用和关注。 首先相比paml，hyphy有以下几大优点：

* 操作使用简单。
* 运行效率更高，支持多线程。
* 模型众多，并且，更新，许多模型都是近几年单独发了篇MBE，虽然我不敢说两个软件谁更好，但Hyphy官方是说比paml更好的，比如hyphy的BUSTED相比paml的branch-site model，hyphy官方说法为

  > [BUSTED](https://stevenweaver.github.io/hyphy-site/methods/selection-methods/#busted) is a method described in [Murrell et al](http://www.ncbi.nlm.nih.gov/pubmed/25701167). It has been extensively tested and shows better power and accuracy than either ["branch-site" models in PAML](http://mbe.oxfordjournals.org/content/24/5/1219.short), or the ["covarion" style models](http://mbe.oxfordjournals.org/content/early/2013/10/16/molbev.mst198).

**接下来介绍的一系列东西，实际上是对Hyphy官方网站的一系列教程的总结，很多东西官网都写得很清楚，官网地址为**[https://www.hyphy.org/。](https://www.hyphy.org/。)

**如果你不想看长篇大论，直接跳到最后的总结部分，那里有最简练的总结**

## 关于Hyphy你需要知道的几件事

**关于Hyphy的不同版本**,hyphy的网页版即是datamonkey，并且还有GUI版本，这里介绍的主要是命令行版本，并且命令行版本也可以分为交互式运行和一行命令运行，这里不介绍交互式方法的使用。

**关于hyphy的安装**,只需要用conda就可以安装了

```text
conda install -c bioconda hyphy
```

**关于hyphy的输入文件**，要求一颗newick格式（只能是此格式）系统发育树以及相对应的fasta序列比对文件（可以是FASTA, phylip, 等等），标注foreground branch，即前景支的方法和paml略微不同，即在newick文件中在分支名和支长（如果有的话）之间加上{Foreground}来标注，或者你可以去hyphy官网的phylotree来在线标注，地址为[http://phylotree.hyphy.org/。](http://phylotree.hyphy.org/。)

**关于多线程支持方面**，2.4.0版本当中，软件中的命令`hyphy`和`HYPHYMPI`已经等同，都是调用多线程，在这个版本之前，`hyphy`调用的是单核，而`HYPHYMPI`则对应的是多线程版本命令。

**关于具体使用方法**，hyphy的使用非常简单，以2.2.0版本为例，如果你要使用多线程命令，则如下有两种方法，分别对应位点模型slac以及指定了前景支的支位点模型absrel,它们都需要openmp支持

```text
# 方法一
HYPHYMPI slac --alignment alignment.fasta --tree tree.newick CPU=interger(需要openmp支持)
# 方法二
mpiexec --oversubscribe -n 10（线程数) HYPHYMPI absrel --alignment alignment.fasta --tree tree.newick --branches Foreground
```

**不同的模型，只需要改相应的模型名称就可以调用了\(替换上面命令的slac或者absrel\)，用法非常简单,如果不特别用branches指定Foreground，那么则会默认对整个系统发育应用模型。**

**关于输出结果及结果可视化**，Hyphy运行的时候，默认打印到屏幕上的结果是以markdown格式输出的，这个结果还是很直观的，而保存到本地文件的结果是以json格式输出的，并不是很直观\(**但json格式可以很方便的用python的json模块提取各种信息，例如pvalue和正选择位点，在多个任务批量操作的时候，非常的方便，这种保存的格式非常具有通用性，其实是件好事**\)，默认是输出到和多序列比对文件相同的文件夹，可以用`--output`来改变输出位置，可以去官网[http://vision.hyphy.org/来可视化输出结果，具体的格式介绍，详见https://www.hyphy.org/resources/json-fields.pdf。](http://vision.hyphy.org/来可视化输出结果，具体的格式介绍，详见https://www.hyphy.org/resources/json-fields.pdf。)

## 重点：模型介绍

关于Hyphy的各种模型，基本上都可以分为不指定foreground和指定foreground运行的两种方式，前者对应的是检测**pervasive \(across the whole phylogeny\) positive or purifying selection**，即整个系统发育中的普遍的正选择/纯化选择，而后者对应的是检测**episodic \(at a subset of branches\) positive or purifying selection**，即检测一部分branches的独立正选择/纯化选择。**所以Hyphy官方经常提到的pervasive selection和episodic selection，意思就是一个指不指定前景，检测整体的选择压力，另一个则是指定前景，检测部分的选择压力。**

### 几大位点模型

#### 适合检测Pervasive selection的位点模型

**①FEL 固定效应似然法\(FEL, Fixed Effects Likelihood\)** 使用最大似然\(ML\)方法来推断每个位点上的非同义\(dN\)和同义\(dS\)替换率，用于给定的编码比对和相应的系统发育。该方法假设在整个系统发育过程中，每个位点的选择压力是恒定的。**注意，FEL适合小到中型数据**

**②SLAC \(Single-Likelihood Ancestor Counting\)** 对于给定的编码比对和相应的系统发育使用最大似然\(ML\)和计数方法的结合来推断每个位点上的非同义\(dN\)和同义\(dS\)替换率。像FEL一样，该方法假设在整个系统发育过程中，每个位点的选择压力是恒定的。**SLAC和FEL精准度相似，但适合更大的数据，并且不适合高度分歧的序列**

**③⭐FUBAR\(Fast, Unconstrained Bayesian AppRoximation\)⭐** 使用贝叶斯方法来推断给定编码比对和相应系统发育的每个位点上的非同义\(dN\)和同义\(dS\)替换率。该方法假设在整个系统发育过程中，每个位点的选择压力是恒定的。**FUBAR适用于中到大数据集，预计在检测位点的普遍选择方面比FEL更有效。FUBAR是推断pervasive selection的首选方法。**

#### 适合检测episodic selection的位点模型

**MEME\(Mixed Effects Model of Evolution\)** MEME\(混合效应进化模型\)采用混合效应最大似然方法来检验个别位点是否受到episodic positive或多样化选择的影响的假设。换句话说，MEME的目的是检测在一定比例的分支下正选择下进化的位点。 对于每个位点，MEME推测两种ω值，以及在给定的分支下，以此ω进化的概率。为了推断ω，MEME会推断α\(dS\) 和两个不同的β\(dN\),β−和β+。在空模型和备择模型中，MEME强制β−≤α。因此β+是空模型和备择模型不同的关键：在空模型中，β+被限制为≤α，但在备择模型中不受限制。最终，当β+&gt;α时，位点被推断为正选择，并使用似然比检验显示显著。

#### 检测蛋白序列比对的正选择/定向选择的位点模型

**FADE\(FUBAR Aproach to Directional Evolution\)** 是一种基于FUBAR引入的贝叶斯框架\(Bayesian framework\)的方法，用来测试蛋白质比对中的位点是否受定向选择的影响。具体地说，FADE将系统地测试，对于比对中的每个位点，与背景分支相比，一组指定的前景分支是否显示对特定氨基酸的替代偏向。该偏差参数的高值表明该位点对特定氨基酸的取代作用大大超过预期。使用贝叶斯因子\(BF\)评估FADE的统计显著性，其中BF&gt;=100提供了强有力的证据，表明该位点正在定向选择下进化。 重要的是，与HyPhy中的大多数方法不同，FADE不使用可逆的马尔可夫模型，因为它的目标是检测定向选择。因此，**FADE分析需要一个有根的系统发育。在使用FADE进行分析之前，可以使用基于浏览器的交互工具“Phylotree.js”来帮助建立树的根。**

### 两个branch-site 模型

#### 检测独立分支的episodic selection\(branch-site model,at a subset of sites\)

**aBSREL \(adaptive Branch-Site Random Effects Likelihood\)** 是常见的“Branch-Site”类模型的改进版本。aBSREL既允许分支的先验指定（即指定foreground branches\)来测试选择，也可以以探索性的方式测试每个谱系以进行选择\(**p-value将自动进行BH校正，为什么叫探索性的方法呢，因为你可以先不指定foreground，来看看哪个支的pvalue更低，然后来针对那一支进行进一步的选择压力分析**）。请注意，探索性的方法将牺牲功效。**⭐aBSREL是在各个分支检测正选择的首选方法⭐，需要注意一点的是，aBSREL是多次独立对指定的每一支进行检验的，也就是说，你指定了许多的branches，实质上和多次指定不同一个branch来多次运行，效果是一样的，而并非将这些branches视为一个整体去做检测**

#### 检测一个基因是否在某一分支或一组分支上的任何位点经历了正选择（即Hyphy官方说比paml的branch site更准的模型（笑😀））

**BUSTED\(Branch-Site Unrestricted Statistical Test for Episodic Diversification\)** 通过测试一个基因是否在至少一个分支的至少一个位点上经历了正选择，BUSTED\(分支位点无限制统计检验\)提供了一个全基因\(非位点特异性\)正选择的测试。当运行BUSTED时，用户可以指定一组前景支来测试正选择\(其余分支被指定为“背景”\)，或者用户可以测试整个系统发生的正选择。在后一种情况下，整个树被有效地视为前景，正选择的检验考虑整个系统发育。这种方法对于相对较小的数据集\(少于10个分类单元\)特别有用，在这些数据集中，其他方法可能没有足够的功效来检测选择。**这种方法不适用于确定有正选择的特定位点。** 对于每个系统发育分区（前景和背景分支位点），BUSTED拟合了一个具有三个速率类的密码子模型，约束为ω1≤ω2≤1≤ω3。与其他方法一样，BUSTED同时估计每个分区属于每个ω类的位点的比例。这种模型作为选择检验中的替代模型，被称为无约束模型。然后，BUSTED通过比较这个模型与前景分支上ω3=1（即不允许正选择）的空模型的拟合度来测试正选择。这个零模型也被称为约束模型。如果零假设被拒绝，那么就有证据表明，至少有一个位点在前景枝上至少有一部分时间经历了正选择。重要的是，一个显著的结果并不意味着该基因是在整个前景的正选择下进化的。

### 选择压力的放松/加强

**RELAX** RELAX是一种假设检验框架，它检测自然选择的强度是否沿着一组指定的测试分支被放松或加强。因此，RELAX不是明确检测正选择的合适方法。相反，RELAX在识别特定基因上自然选择严格程度的趋势和/或变化方面最有用。K&gt;1表示选择强度加强，K&lt;1表示选择压力放松。 RELAX需要一组指定的 "测试 "分支与第二组 "参考 "分支进行比较（注意，不必分配所有的分支，但测试集和参考集各需要一个分支）。RELAX首先对整个系统发育过程拟合一个具有三个ω类的密码子模型（空模型）。然后，RELAX通过引入作为选择强度参数的参数k（其中k≥0）作为推断ω值的指数：ωk来测试放松/强化选择。具体来说，RELAX固定推断的ω值（都是ωk），并对测试分支推断出一个将比率修改为ωk的k值（替代模型）。然后，RELAX进行似然比检验，比较替代模型和空模型。

## 总结

用法来说，以我用的2.2.0版本为例子，（2.4.0直接用hyphy命令即可）

```text
HYPHYMPI slac/absrel/fubar(指定运行的模型名称) --alignment alignment.fasta(序列比对文件) --tree tree.newick(树文件) --branches Foreground（此项可省略，则不指定前景) CPU=interger(线程数，需要openmp支持)
```

模型上来说： 如果你要检测类似paml中的M8位点模型，最好用FUBAR，如果是小数据，则用FEL，大数据并且分歧度不是很高用SLAC。 如果你要检测某个前景支当中正选择位点，最好用MEME。 如果你要检测单独的某个branch是否存在正选择，最好用aBSREL。 如果你要检测一系列的branches的正选择，即检验你的这个基因，在指定的branches的任意一个位点是否在某段时间经历过正选择，则用BUSTED，BUSTED是不适合检测单独位点的正选择的。 如果你要检测选择压力的放松/加强，用RELAX。 如果你要用蛋白序列来检测氨基酸位点正选择/定向选择，用FADE。

最后再提一下，几乎所有模型\(还有一些没常用的模型没有提到\)都可以分为指定前景和不指定前景的模式运行，但不是都适合，就像官方说的那样，根据你的目的不同，会有最优选择，当然你也可以把某种模型都跑一遍，比如各种位点模型都走个流程，并且你也可以结合paml的模型，例如，对于检测Pervasive selection的位点模型，你可以结合paml的M8、M2a来分析。对于检测episodic selection的branch-site，你可以结合paml的branch-site modelA和BUSTED/aBSREL来比较分析。

**以上的所有文字，都是笔者根据官方以及一些文献当中对于hyphy的使用总结、翻译，如有错误使用之处，还请各位多多指正。**

