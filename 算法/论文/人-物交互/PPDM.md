# PPDM: Parallel Point Detection and Matching for Real-time Human-Object Interaction Detection[精读]
## 摘要

作者提出单阶段的HOI检测方法，表现SOTA。这是第一个实时的HOI检测方法。传统的的HOI检测方法由两个阶段组成，但是它的有效性和效率受到顺序和独立架构的限制。文中作者提出PPDM的HOI检测框架。在PPDM中，HOI被定义为一个point triplet<human point,interaction point,object point>，其中human point和object point是检测框的中心，interaction point是human point和object point的中点。

PPDM包含了两个并行分支，也就是**点检测分支**和**点匹配分支**。其中点检测分支预测是三个点，点匹配分支预测从interaction point到对应的human point和 object point的偏移。如果human point和object point是来自同一个interaction point，则认为它们是匹配的。

在作者新颖的并行框架中，interaction point 隐式地为人和物的检测提供了上下文和正则化。抑制孤立的检测boxes，因为它不可能形成有意义的HOI triplets(个人理解：人-物候选区单独产生，没有考虑到他们之间的联系，这种情况不利于检测，所以要抑制)，这增加了HOI的检测精度。更何况人和物检测的boxes只是应用在数量有限并过滤过的候选interation point,节省了大量计算消耗。此外，作者了建立了一个新的数据集HOI—A。

## 1. Introduction

传统的HOI方法由两个阶段组成。第一个阶段是人-物候选区检测。这阶段可以得到很多大量的人-物对候选区(M×N)。第二阶段是预测每个人-物候选区的交互。这种两个阶段方法的有效性和效率受到顺序性和独立性的限制。候选区的产生阶段完全基于对象检测的置信度。**每个人/物候选去单独产生。组合两个候选区形成有意义的HOItriplet的可能性在第二阶段并没有考虑**(个人理解：就是摘要中所说到的受到到独立架构的限制)。所以，产生的人-物候选区可能质量较低，并且在第二阶段，所有人-物候选区需要线性扫描，开销很大。所以作者认为需要非顺序性的和高耦合度的框架。

PPDM的第一个分支估计中心点(interation,human和object point)，对应大小，和两个局部偏移(human和object point)的点检测。因为interaction point可以认为给人和物的检测提供上下文信息，也就是说，对interation point的估计可以隐式地增强人和物的检测(个人理解：交互点的估计需要增加感受野，因为需要人和物的信息，所以感受野的增大也有利用为人和物的检测)。第二个分支是点匹配，估计interation point到human point和object point的偏移。

作者贡献有三：（1）把HOI检测任务视为点检测和点匹配问题，并提出单阶段的PPDM。(2)PPDM是第一个在HOCI—DET和HOI—A benchmark中达到实时并表现SOTA的的HOI检测方法。(3)HOI-A

## 2. Related Work

略略略....

## 3. Parallel point dection and matching

### 3.1 Overview

![](https://gitee.com/weifagan/MyPic/raw/master/img/ppdm.png)

图3.作者首先应用keg-point heatmap预测网络来提取提取特征，如Hourglass-104 or DLA-34。a) **Point Detection Branch**:基于提取的视觉特征，作者利用三个卷积模块来预测heatmap中的交互点，人中心点和物中心点，此外，回归的2-D size和人和物的局部偏移来产生最后的box。b) **Point Matching Branch**:此分支的第一步是分别回归从交互点到人中心点到物中心点的偏移。基于预测的点和位移，第二步是每一个交互点匹配人中心点和物中心点来产生一系列的tirplets。

### 3.2 Point Detection

图3中输入图像是$I$,经过特征提取器产生的特征$V$。人中心表示为$(x^h,y^h)$,其对应的大小为$(w^h,h^h)$，局部偏移量为$\delta c^h$，弥补输出步幅引起的离散化误差。GT人中心点对应的低分辨率点(heatmap产生)为$(\overline{x}^h,\overline{y}^h)=(\frac{x^h}{d},\frac{y^h}{d})$的向下取正。

**Point location loss.** 直接检测点比较困难，所以作者使用关键点估计方法将点映射到高斯核热图中。所以点检测转换为heatmap估计任务。三个GT低分辨率的点分别映射到三个高斯heatmap，包括人中心点heatmap $\overline{C}^h$,物中心点heatmap $\overline{C}^o$,交互点heatmap  $\overline{C}^a$,其中 $\overline{C}^o$和$\overline{C}^a$是多通道的。在特征映射$V$上，分别添加三个卷积网络来产生三个heatmap。loss 函数为：

![](https://gitee.com/weifagan/MyPic/raw/master/img/PPDM_loss.png)

**Size and offset loss**.四个卷积模块添加到特征映射$V$来分别产生人和物的产生2-D size和局部偏移。$L_{off}$为

![](https://gitee.com/weifagan/MyPic/raw/master/img/ppdm_loss_off.png)

### 3.3 Point Matching

偏移分支有两个卷积模块组成。

**Diaplacement loss**:

![](https://gitee.com/weifagan/MyPic/raw/master/img/ppdm_loss_dis.png)

**Triplet matching**: 判断人中心点和物中心点是否匹配看两个方面，一是交互点加上偏移后，靠不靠近大概的人/物的中心点，二是有高的置信度。

![](https://gitee.com/weifagan/MyPic/raw/master/img/ppdm matching.png)

### 3.4 Loss and Inference

最后的loss为：

![](https://gitee.com/weifagan/MyPic/raw/master/img/ppdm loss final.png)

在推理阶段，作者首先在预测的人、物和交互点的heatmap上用一个3x3 max-pooing操作，然后通过对应的置信度选择top K个人中心点，物中心点和交互点，最后triplets匹配。对于每个匹配的人中心点，最后得到的box为：

![](https://gitee.com/weifagan/MyPic/raw/master/img/ppdm box final.png)

## 4 个人总结

**1.文章解决什么问题**:

解决传统的两阶段HOI检测问题。

**2.用自己的话阐述文章思路**

作者提出并行的单阶段的HOI检测网络，PPDM。PPDM首先用key-point heatmap预测网络来提取特征，然后有两个并行分支，分别是点检测分支和点匹配分支。在点检测分支中，预测三点（人中心点、物中心点、交互点）基于对应大小，以及局部偏移。在点匹配分支中，预测交互点到人中心点和物中心点的偏移，根据置信度选取TOP K个人中心点、物中心点和交互点，最后匹配triplets。

**3.关键因素**

* 直接预测点比较困难，所以将点映射到高斯核热图中，将点检测转换为 heatmap估计任务。
* 传统的HOI检测是顺序性的两个阶段，先候选区检测再是预测交互，而PPDM则是并行分支。一个分支预测人-物box及其交互点，另一个分支则预测交互点和人-物中心点的偏移。
* 传统的HOI检测人-物检测是单独，没有考虑到他们之间的联系，而PPDM则是人中心点-交互点-物中心点一起估计，为了更好地检测交互点，增加感受野，感受野中带有人-物的上下文信息，这考虑到了它们之间的联系。

**4.为我所用**

* 通过key-point heatmap网络，将直接点预测转换为在heatmap上预测。
* PPDM的并行分支分别负责不同的任务。