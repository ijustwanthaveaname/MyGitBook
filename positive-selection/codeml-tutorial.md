---
description: 本教程根据文件以及Paml官方教程所述而写，如有错误之处请联系作者改正，谢谢！
---

# Codeml tutorial

## 安装

可以通过conda直接安装

```text
conda install paml
```

## 输入文件

* Paml的codeml需要newick格式树，标记一个branch为前景支可以用\#1, \#2...\#n，标记整个clade为前景可以用$1, $2...$n, 相同数字标记的branch/clade有着同样选择压的假设。具体标记如下所示:

```text
((a #1, b), c) # 标记a的branch作为foreground
((a, b) $1, c) # 标记a,b以及a和b的祖先分支作为foreground
```

* 密码子比对的序列文件\(phylip格式\)，可以用MACSE比对，然后Gblocks过滤后，用prank转化格式（可以转格式的软件有一万种，随便选一个或者自己写都可以）。
* codeml.ctl配置文件

## 模型使用

### site-model

用于检测pervasive positive selection的位点模型有：

* M0\(one ratio\)：**在codeml.ctl中设置NSsites=0。**此模型计算整棵树的平均ω。
* M1a\(neutral\): **在codeml.ctl中设置NSsites=1。**此模型假定整棵树有两种不同比例的ω，比例分别为p0, p1=1-p0, 对应ω0&lt;1, ω1=1。
* M2a\(selection\): **在codeml.ctl中设置NSsites=2。**此模型假设整棵树有三类不同比例ω，比例分别为p0, p1, p2=1-p0-p1,分别对应ω01。
* M3\(discrete\): **在codeml.ctl中设置NSsites=3。**此模型假定整棵树有3种不同比例的ω，比例分别为p0, p1, p2=1-p0-p1,分别对应ω0, ω1, ω2，每个ω都不设限制。
* M7\(beta\): **在codeml.ctl中设置NSsites=7。**此模型假设不同位点的ω属于\(0, 1\)矩阵，并呈β曲线分布，一共有10类位点，每个位点比例相同，但ω不同。
* M8\(beta&ω\): **在codeml.ctl中设置NSsites=8。**此模型在M7的基础上多了一个位点分类，即前10个ω比例相同但ω大小不同，而最后一个位点固定ω&gt;1。**对应的null模型为M8a，指定方法见下面**。

可以一次运行上述所有模型，参考配置如下:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = mlc * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 0 * 设置各个branch的ω相同(因为位点模型只考虑位点间的变异)
       NSsites = 0 1 2 3 7 8 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 0
         omega = .8 * 可以指定多个omega作为起始值，来看结果是否准确
         getSE = 0
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

而M8a模型，需要用fix\_omega =1，和omega = 1来固定：

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = M8a * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 0 * 设置各个branch的ω相同(因为位点模型只考虑位点间的变异)
       NSsites = 8 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 1
         omega = 1 * 可以指定多个omega作为起始值，来看结果是否准确
         getSE = 0
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

**用于比较的模型有**：

* M3\(离散\) vs M0\(单一ω\) （**作者不推荐用此来检测正选择位点, 注意M3由于没有限制ω大小，所以并不一定会有ω&gt;1的情况发生，即使似然比检验显著了也只能说明整棵树的位点不符合M0对应的单一ω情况**\)
* M2a\(正选择\) vs M1a
* M8\(正选择\) vs M7
* M8\(正选择\) vs M8a

**似然比检验方法**

我们只需要找到结果文件中对应每个模型的lnL那行，记录下每对模型的lnL，以及np（参数数目\)，然后我们可以用R语言做卡方检验来检验每对结果，例如如果M1a对应的lnL为-18260.863931，np为20，M2a对应的 -18257.608795, np为22，则在R中检验（检验方式也有一万种，你也可以用paml自带的卡方检验功能，或者excel也都可以）如下:

```r
pchisq(2*(-18257.6088--18260.86393), df=22-20, lower.tail=F)
```

用paml自带的chi2命令（zsh中运行\):

```text
chi2 $((22-20)) $((2*(-18257.6088+18260.86393)))
```

**注意，正选择位点的后验概率有NEB和BEB两种计算方法，而作者推荐忽略NEB，使用BEB，一般来说我们认为后验概率≥95％的位点比较可靠，BEB只在M2a和M8下运行。**

### branch model

branch model与clade model类似，但是只包含了单一比例的位点，因此检测的是分区之间的平均差异，这种方法是敏感度较低的方法，但是在检测选择压力的放松方面会特别有用。

由于不考虑位点间的差异，因此需要用NSsites = 0来固定。

主要有下述三种模型：

* free ratio: 通过model = 1, NSsites = 0来指定，它假定每个分支都有一个独立的ω比率，这个模型参数非常丰富，**作者不推荐使用**
* two ratio: 通过model = 2, NSsites = 0来指定，model = 2允许你有几个ω比率。你必须指定有多少个比率和树文件中用分枝标签来标定哪个分枝对应哪个比率。
* one ratio/M0: 通过model = 0, NSsites = 0来指定，实质上就是位点模型里的M0。

我们通过比较 two ratio和one ratio/M0的结果，进行似然比检验即可，参考配置如下，M0参考位点模型。

**two ratio**:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = tworatio * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 2 * 设置各个branch的ω有2个或以上不同，对应指定的前景背景支
       NSsites = 0 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 0
         omega = 2 * 可以指定多个omega作为起始值，来看结果是否准确
     fix_alpha = 1
         alpha = 0
        Malpha = 0
         getSE = 0
         ncatG = 4
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

### branch-site model

**现在的支位点模型一般指代branch site ModelA，通过model = 2, NSsites = 2来指定,对应的空模型ModelAnull通过fix\_omega = 1，omega=1固定ω为1来指定。**branch-site 模型被设计用来检验一系列独立分支的独立正选择事件（尽管它也能在进化支和混合的分区上应用的很好）。不同于branch和clade model，branch-site model的前景和背景支分区有着明显的区别，它允许4类位点:

* 0：在所有分支上0&lt;ω0&lt;1。
* 1：在所有分支上ω1=1。
* 2a: 前景支中ω2a=ω2b≥1，背景支中0&lt;ω2a=ω0&lt;1。
* 2b:前景支中ω2b=ω2a≥1， 背景支中ω2b=ω1=1。

**这个模型能有效的检测独立的正选择事件，但是会导致假阳性的结果，尤其当正选择事件也发生在背景支时。**

参考配置：

**ModelA**:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = ModelA * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 2 * 设置各个branch的ω有2个或以上不同，对应指定的前景背景支
       NSsites = 2 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 0
         omega = 2 * 可以指定多个omega作为起始值，来看结果是否准确
     fix_alpha = 1
         alpha = 0
        Malpha = 0
         getSE = 0
         ncatG = 4
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

**ModelAnull**:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = ModelA * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 2 * 设置各个branch的ω有2个或以上不同，对应指定的前景背景支
       NSsites = 2 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 1
         omega = 1 * 可以指定多个omega作为起始值，来看结果是否准确
     fix_alpha = 1
         alpha = 0
        Malpha = 0
         getSE = 0
         ncatG = 4
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

### clade model

**进化支模型主要用来检测歧化选择/趋异选择\(当然也可以用来检测正选择事件\)。**

主要有下述两种模型：

* Clade model C\(简称CmC\):**通过model = 3, Nssites = 2, ncatG = 3来指定。**CmC假定两类位点在整个系统发育是保守的，即0&lt;ω00以及ωD1≠ωD2&gt;0。**空模型为M2a\_rel， 通过model = 0, Nssites = 22指定。**
* Clade model D\(简称CmD\):**通过model = 3, Nssites = 3, ncatG = 3来指定。**CmD和CmC类似，但是三类位点\(ω0, ω1, ωD\)都不设限制。**空模型为M3。**

参考配置：

M2a\_rel:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = M2a_rel * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 0 * 设置各个branch的ω相同(因为位点模型只考虑位点间的变异)
       NSsites = 22 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 0
         omega = .8 * 可以指定多个omega作为起始值，来看结果是否准确
         getSE = 0
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

CmC:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = CmC * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 3 * 设置各个branch的ω有2个或以上不同，对应指定的前景背景支
       NSsites = 2 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 0
         omega = 2 * 可以指定多个omega作为起始值，来看结果是否准确
     fix_alpha = 1
         alpha = 0
        Malpha = 0
         getSE = 0
         ncatG = 3
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

CmD:

```text
       seqfile = alignment.phy * 序列比对文件
      treefile = tree.newick * 树文件
       outfile = CmD * 输出文件
         noisy = 9 * 0,1,2,3,9：输出多少信息在屏幕上
       verbose = 0 * 0表示简洁输出
       runmode = 0 * 使用用户提供的树
       seqtype = 1 * 1指codons序列
     CodonFreq = 2 * 使用F3X4计算密码子频率
         clock = 0 * 不设置分子钟
        aaDist = 0 * 氨基酸间距离相等
         ndata = 1 * 输入的多序列比对的数据个数
         model = 3 * 设置各个branch的ω有2个或以上不同，对应指定的前景背景支
       NSsites = 3 * 对应上述
         icode = 0 * 0表示通用密码子, 1表示哺乳类线粒体密码子， 2-10参见官网
     fix_kappa = 0
          kappa = 2
     fix_omega = 0
         omega = 2 * 可以指定多个omega作为起始值，来看结果是否准确
     fix_alpha = 1
         alpha = 0
        Malpha = 0
         getSE = 0
         ncatG = 3
  RateAncestor = 0 * 若设置为1则会计算祖先序列，会在结果文件rates中给出各个位点的碱基替换率
    Small_Diff = .5e-6
     cleandata = 1 * 删除模糊位点
        method = 0
   fix_blength = 0 * 忽略支长信息
```

