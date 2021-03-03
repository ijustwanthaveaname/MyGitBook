---
description: >-
  当有了最适模型以及过滤好的比对质量较高的序列比对结果，我们就可以进行系统发育树的构建，系统发育树构建的方法有很多，当今流行的两种统计学方法是最大似然法(ML)和贝叶斯推断(BI)方法构树，下面将介绍两种方法的主流软件使用。
---

# Step4.Building a phylogenetic tree

## **iqtree（易用、快速、最新、ML法构树、同时可以寻找最优构树进化模型）**

-b和-bb分别对应的是普通的bootstrap和快速bootstrap方法，后者会更快，但精确度会低些，下面是两条常用命令，-s指定输入序列，-m里MFP表示的是自动搜索最适模型然后进行构树，同样你也可以自己指定一个模型。

`iqtree -s input.aln -m MFP -b 100(快速)/1000(慢速) -nt AUTO`

`iqtree -s input.aln -m MFP -bb 1000(快速bootstrap) -bnni`

## **raxml\(权威、精准、广泛使用、快速、ML法构树、用法复杂\)**

`raxmlHPC -s seqname -n xxx -m GTRGAMMAI -T 40 -N 1000  -p 12345 -f a -x 20170808`

 -s                 输出用于建树的序列名字。

 -n                 定义输出的树文件的名字。

 -m                这是模型设置，具体的模型要看你的是什么模型，上面仅仅是示例。

 -T                 设置线程，前提是你的服务器支持。

 -N                设置自举次数，一般论文要求1000。

 -p\(小写\)        一个用于简约推断的随机数，随机数可以自己设。

 -f a               快速自举分析

 -x\(小写\)        指定一个整数（随机数，可以随便取比如12345）并启用快速bootstrapping。

 -k（小写）    指定bootstrapped树应该输出分支长度。默认值是关闭的。

## **Mrbayes（准确、精准、广泛使用、权威、配置较为复杂、速度慢）**

输入文件格式要求必须为nexus格式，用法分为命令交互配置参数或者直接**在nexus数据文件末尾配置参数。**推荐直接配置好参数，方便多次运行。

### 常用参数配置（在nexus文件最末尾添加\)

begin mrbayes;

lset Nst=6 rates=invgamma;

\#lset表示配置模型参数，Nst=1为JC69、2为F81、6为GTR;rates可以为gamma/invgamma/adgamma/kmixture/equal/lnorm/propinv/，上述配置及为最为常用的GTR+Gamma+I方法

mcmcp ngen=2000000 samplefreq=100 printfreq=1000 nchains=4 nruns=2 burninfrac=0.25;

mcmcp savebrlens=yes checkpoin=yes checkfreq=5000;

\#其余参数不用动，需要注意的是ngen和samplefreq，ngen表示总的代数，samplefreq表示抽样频率，ngen除以samplefreq等于产生的样本数，最终收敛诊断时，需要根据是否收敛来增加ngen或减少samplefreq

mcmc;

sump;

sumt contype=allcompat relburnin=yes burninfrac=0.25;

end;

### **Linux下使用和安装多线程mrbayes（ubuntu版本为例）**

#### 安装必备库和软件

`sudo apt-get install automake autoconf pkg-config autoconf-archive gcc g++ make`

#### 下载mpi

`wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.5.tar.gz`

#### 解压缩及安装mpi

`tar -xzvf ./openmpi-1.10.1.tar.gz`

`cd openmpi-1.10.1`

`./configure --prefix=/usr/local/mpi`

`sudo make -j` 

`sudo make install`

`sudo echo ‘export PATH=/usr/local/mpi/bin:$PATH’ > >~/.bashrc`

`sudo echo ‘export LD_LIBRARY_PATH=/usr/local/mpi/lib:$LD_LIBRARY_PATH’ >> ~/.bashrc`

`source ~/.bashrc`

#### 下载最新版本Mrbayes

`wget` [`https://github.com/NBISweden/MrBayes/releases/download/v3.2.7/mrbayes-3.2.7.tar.gz`](https://github.com/NBISweden/MrBayes/releases/download/v3.2.7/mrbayes-3.2.7.tar.gz)\`\`

#### 解压缩及安装

`tar -xzvf ./mrbayes-3.2.7.tar.gz`

`cd ./mrbayes-3.2.7.tar.gz`

`sudoautoreconf -i`

`./configure --with-mpi --with-beagle=no`

`sudo make -j`

`sudo cp ./src/mb /usr/local/bin`

#### 多线程使用Mrbayes

`mpirun -np 10(线程数) -oversubscribe mb xxx.nex #注！链的总数必须能被MPI处理器的数目均匀地整除。`

**跑完之后用tracer拖进去.p结尾的两个文件然后看是否ess都大于200，不行的话就要改代数和抽样频率重新跑。**

