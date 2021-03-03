---
description: 请记住，beast有beast1和beast2两个版本，本文介绍的主要是最新的beast2版本的使用。
---

# Beast2

首先记住两个时间校准记录的网站：[http://www.timetree.org/](https://links.jianshu.com/go?to=http://www.timetree.org/%22%20\t%20%22https://www.jianshu.com/p/_blank) 、[https://fossilcalibrations.org/](https://links.jianshu.com/go?to=https://fossilcalibrations.org/%22%20\t%20%22https://www.jianshu.com/p/_blank) ，根据你研究的类群，从网站上搜集相应时间校准点。

## **beast2主要使用\(分为tip dating和node dating\)**

需要先用beauti配置xml文件，再用beast运行xml。通过配置不同的位点模型\(Site model\)、分子钟模型（Clock model）和树先验（Tree prior）模型可以获得不同的组合。但对于一个给定的数据集，最适的核苷酸替换模型可以通过jModelTest之类的模型选择工具可以确定，而选择最适的分子钟模型（Clock model）和树先验（Tree prior）一般通过贝叶斯因子法（BF）比较不同模型的LnL值后确定。

**MCMC部分设置:**这部分的设置总的来说，因不同数据而异，“总代数=3000 x 序列条数的平方”这个公式仅仅只是粗略地估算（请注意这句话）。

**但是，最主要的一点需要保证最后的样本最少得有10,000个。**

 **样本数量=Length of chain（总代数） ÷ Log parameters every（采样频率）**

运行链长：10000000（默认1千万步，根据自己实际情况自行调整）

tracelog：链长运行2000次抽样一次

screenlog：运行10000次打印到屏幕一次

treelog.t:tree：运行2000次打印一个拓扑结构树

结果主要生成3个文件：日志文件（\*.logs）、树文件\(\*.trees\)、xml状态文件\(\*.state\)，其中日志文件.log文件即将用来进行后面 模型的评估，树文件用来进行展示。

完成后用TreeAnnotator.exe生成树。

**注：一般来说我们做dating都是给定树的情况下，而beast是边建树边算时间，所以要想这样做必须要修改xml文件，方法步骤如下：**

①BEAUti中设置operator weight=0

②删除xml中的元素

* Wide-exchange, which are elements with spec="Exchange" and isNarrow="false"
* Narrow-exchange, elements with spec="Exchange" and isNarrow not specified
* Wilson-Balding, elements with spec="WilsonBalding"
* Subtree-slide, elements with spec="SubtreeSlide"

③修改xml中的随机树文件配置，改为你的固定树：

如原本为：&lt;init id="RandomTree.t:firsthalf" spec="beast.evolution.tree.RandomTree" estimate="false" initial="@Tree.t:firsthalf" taxa="@firsthalf"&gt;

 &lt;populationModel id="ConstantPopulation0.t:firsthalf" spec="ConstantPopulation"&gt;

 &lt;parameter id="randomPopSize.t:firsthalf" spec="parameter.RealParameter" name="popSize"&gt;1.0&lt;/parameter&gt;

 &lt;/populationModel&gt;

 &lt;/init&gt;

修改为：&lt;init spec="beast.util.TreeParser" id="NewickTree.t:firsthalf" initial="@Tree.t:firsthalf" taxa="@firsthalf" IsLabelledNewick="true" adjustTipHeights="true" newick="\(Tarsius\_syrichta:0.4936231124,Lemur\_catta:0.3372111528,\(\(\(\(\(\(Homo\_sapiens:0.0491166021,Pan:0.0597029518\):0.0252656155,Gorilla:0.0584496136\):0.0759288710,Pongo:0.1373848367\):0.0480111510,Hylobates:0.1693883784\):0.1156944165,\(\(\(Macaca\_fuscata:0.0160477266,M\_mulatta:0.0225824745\):0.0341163564,M\_fascicularis:0.0558327688\):0.0428505688,M\_sylvanus:0.0713537499\):0.2445849034\):0.1112770850,Saimiri\_sciureus:0.4437750830\):0.2592669805\);"&gt;

 &lt;/init&gt;

**然后tracer打开log文件，看是否收敛，不收敛调代数和抽样频率继续跑。**

**最后用Treeannotator生成最终时间树文件，命令行可以用**

`treeannotator -burnin 10 -limit 0.5 -heights ca -target user.tree xxx.trees outlinux.tree`

### **beagle（beast的一个库，可以使beast支持多线程）**

windows:https://github.com/beagle-dev/beagle-lib/releases/download/v3.1.0/BEAGLE.v3.1.0.msi

linux:https://github.com/beagle-dev/beagle-lib/archive/master.zip

linux安装见：[https://zhuanlan.zhihu.com/p/65468152](https://zhuanlan.zhihu.com/p/65468152)

