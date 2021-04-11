# FaceBoxes: A CPU Real-time Face Detector with High Accuracy

## 1. 论文要解决什么问题？

要保持高精度，还要在CPU上达到实时？还真有点难，但是Shifeng Zhang等人针对这个问题，提出了人脸检测模型FaceBoxes，表现SOTA。

## 2. FaceBoxes如何解决问题？

FaceBoxes框架如图1所示，主要包括Rapidly Digested Convolutional Layers (RDCL)和Multiple Scale Convolutional Layers (MSCL)模块，还有anchor密集策略。

![img](![](https://gitee.com/weifagan/MyPic/raw/master/img/faceboxes.PNG)

### 2.1 RDCL

RDCL的目的是为了快速下采样，让模型能够在CPU上面能达到实时。RDCL采用的方法是缩小空间大小，选择合适的卷积核大小和减少输出通道。

- 缩小空间大小：Conv1, Pool1, Conv2 and Pool2 的步长分别是4, 2, 2和 2, 空间大小快速降低了32倍。
- 合适的卷积核大小：前几层的一些核应该是比较小，以便加速，但是也应该足够大，以减轻空间大小减小而带有的信息丢失(为什么可以减少信息丢失s)。Conv1, Conv2的核大小为7x7, 5x5，所有池化层的核大小为3x3。
- 减少输出通道：使用C.ReLU减少输出通道，操作如图2(a)所示。因为C.ReLU作者统计发现底层卷积时卷积核存在负相关，也就是说假设我们本来使用10个卷积核，但是现在只需要用5个卷积核，另外5个卷积核的结果可以通过负相关得到。结果表明使用C.ReLU加速的同时也没损失精度。

![img](![](https://gitee.com/weifagan/MyPic/raw/master/img/faceboxes1.PNG)

### 2.2 MSCL

MSCL是为了得到更好地检测不同尺度的人脸。

* 深度：在MSCL模块中，随着网络的加深，便得到不同大小的特征映射(多尺度特征)。在不同大小特征映射中设置不同大小的anchor，有利于检测不同大小的人脸。
* 宽度：Inception由多个不同核大小的卷积分支组成。在这些分支中，不同的网络宽度，也有不同大小的特征映射。通过Inception，感受野也丰富了一波，有利用检测不同大小的人脸。

MSCL在多个上尺度进行回归和分类，在不同尺度下检测不同大小的人脸，能够大大提高检测的召回率。

### 2.3 Anchor密度策略

Inception3的anchor大小为32，64和128，而Conv3_2和Con4_2的anchor大小分别为256，512。anchor的平铺间隔等于anchor对应层的步长大小。例如，Con3_2的步长是64个像素点，anchor大小为256x256，这表明在输入图片上，每隔64个像素就会有一个256x256的anchor。关于anchor的平铺密度文中是这样定义的：
$$
A_{density}=A_{scale}/A_{interval}
$$
其中$A_{density}$和$A_{interval}$分别为anchor的尺度和平铺间隔。默认的平铺间隔(等于步长)默分别认为32，32，32，64和128。所以Inception的平铺密度分别为1,2,4，而Con3_2和Con4_2的平铺密度分别为4,4。

可以看出来，不同尺度的anchor之间存在平铺密度不平衡的问题，导致小尺度的人脸召回率比较低，因此，为了改善小anchor的平铺密度，作者提出了anchor密度策略。为了使anchor密集n倍，作者均匀地将$A_{number}=n^2$个anchor铺在感受野的中心附近，而不是铺在中心，如图3所示。将32x32的anchor密集4倍，64x64的anchor密集两倍，以保证不同尺度的anchor有相同的密度。

![image-20201121200018610](https://gitee.com/weifagan/MyPic/raw/master/img/faceboxes.JPG)

### 2.4 训练

**数据扩增**：

* 颜色扭曲
* 随机采样
* 尺度变换
* 水平翻转
* Face Boxes过滤：经过数据扩增的图片中，如果face boxes的中心还在图片上，则保留重叠部分，然后把高或者宽<20的过滤掉(这个操作不是很懂了，为什么把小目标过滤掉？)

**匹配策略**：训练期间，需要确定哪些anchor对应脸部的bounding box，我们首先用最佳jaccard重叠将每一张脸匹配到anchor，然后将anchor匹配到jaccard重叠大于阈值的任何一张脸。

**Loss function**: 对于分类，采用softmax loss，而回归则采用smooth L1 损失。

**Hard negative mining**：anchor 匹配后，发现很多anchor是负的，这会引入严重的正负样本不平衡。为了快速优化和稳定训练，作者对loss进行排序然后选择最小的，这样子使得负样本和正样本的比例最大3:1。

## 3 实验结果如何？

**Runtime**

![image-20201121212045119](https://gitee.com/weifagan/MyPic/raw/master/img/faceboxes1.JPG)

**Evaluation on benchmark**

在FDDB上SOTA。

## 4.对我们有什么指导意义？

* 要在CPU上达到实时，考虑一开始就对特征进行快速下采样。
* 在浅的卷积层考虑使用CReLU，可以减少计算量。
* MSCL告诉我们，检测各种不同大小的物体，考虑从深度和宽度上丰富感受野。
* anchor密度策略告诉我们考虑anchor密度以提高召回率。

# FootAndBall: Integrated player and ball detector

## 1.论文要解决什么问题？

要在高分辨率、远距离的视频中检测出足球运动员和足球。

## 2.所提算法如何解决问题？

### 2.1 检测难点

* 足球
  * size小
  * 容易模糊变形
* 运动员
  * 遮挡
  * 异常姿态
  * 相对于足球容易检测

### 2.2 FootAndBall

### 2.2.1 输入输出

**FootAndBall输入**：1920x1080

**FootAndBall主要有三个输出:**

* ball confidence map(对足球出现在某个格子的概率进行编码)
* player confidence map(对运动员出现在某个格子概率进行编码)
* player bounding boxes tensor(对运动员bounding boxes进行编码)

### 2.2.2 网络结构

FootAndBall使用了**FPN**(feature pyramid network，特征金字塔网络)思路，结合低级特征和高级特征，改善大背景下的小目标的检测。网络框架如图4所示。

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/footandball.PNG" style="zoom: 80%;" />

由图4可知，较高层的卷积块上采样后与经过1x1卷积较低层的特征进行相加，输出三个head。其中Ball confidence map在比较浅的层输出，因为球比较小，也不需要太大的感受野。

由图4可知，input image宽和高分别是w,h，Ball confidence map的大小为(w/4,h/4,1)。Ball confidence map的位置$(i,j)$对应回原图$(x,y)=(\lfloor k(i-0.5),k(j-0.5) \rfloor$)，其中$k=4$。Player confidence map同理。

Player bounding boxes 编码为$(x_{bbx},y_{bbx},w_{bbx},h_{bbx})$。其中$(x_{bbx},y_{bbx})$为confidence map上对应grid ceil中点的相对位置，而$w_{bbx},h_{bbx}$为bbxes在confidence map上的宽高。

现在设置Player confidence map输出坐标为$(i,j)$，即预测出player坐标为$(i,j)$，则bounding box中心在原图上的坐标为$(x'_{bbx},y'_{bbx})=(\lfloor k(i-0.5)+x_{bbx}w \rfloor,\lfloor k(j-0.5)+y_{bbx}h \rfloor)$，其实player的预测坐标加上预测的偏移量。



### 2.2.3 损失函数

损失函数主要三个成分：ball classification loss, player classification loss and player bounding box loss。计算这三个成分都是使用二值交叉熵函数。

**ball classification loss**:

![](https://gitee.com/weifagan/MyPic/raw/master/img/footandball1.PNG)

其中，$c_{ij}^{B}$是ball confidence map上$(i,j)$位置上的值，${Pos}^{B}$是ball confidence map上GT的位置集合(GT位置的8邻域也要算上该集合)。${Neg}^B$是负样例集合，是ball confidence map上没有GT的位置集合。

**player classification loss**：

![](https://gitee.com/weifagan/MyPic/raw/master/img/footandball2.PNG)

${Pos}^{B}$没有算上GT位置的8邻域，因为feature map的size比较小。

**player bounding box loss**:

![](https://gitee.com/weifagan/MyPic/raw/master/img/footandball3.PNG)

其中$I_{(i,j) \in R^4}$是featue map位置$(i,j)$上预测的bounding box，而$ g_{(i,j)}$ 是对应的GT。

**loss**：

![](https://gitee.com/weifagan/MyPic/raw/master/img/footandball4.PNG)

因为正样例和负样例数量不平衡，所以要限制一下，正负比例最多1:3。

## 3 实验结果如何？

* 1920 x 1080 ：37 FPS

## 4 对我们的指导意义？

FootAndBall 主要使用全卷机网络对目标进行预测，并结合FPN的思想。

* 对于小物体检测，考虑FPN
* 可以使用全卷积在feature map上做预测。



# Objects as Points

## 1.论文要解决什么问题？

目标检测将图像的目标识别为轴对称的boxes，很多目标检测器几乎会列出的潜在对象位置，并对每个位置分类，这是很浪费时间，没有效率和需要额外的后处理。因此，作者提供一个更简单有效的方案，将目标建模为一个点-边框中心点，使用关键点估计寻找中心点和回归其他对象属性。例如大小，3D位置，方向甚至是姿态。

## 2.所提算法如何解决问题？

文中作者提供了一个更简单更高效选择。作者将一个对象表示为边框中心的一个点，其他性能，例如对象大小，维度，3D扩展，方向和姿态等则可以**从图像特征的中心位置直接回归**。这样子的话目标检测变成一个是标准的关键点检测问题。作者简单地把输入图像喂入全卷基层然后产生一个heatmap。在这张heatmap中的峰值对应对象的中心。在每个峰值的图像特征预测对象边框的高和宽。模型使用标准的监督学习训练。推理是单个网络的前向传播，没有nms的后处理。

总结一下这个过程：输入图片->全卷积->heatmap(峰值)->预测宽高

### 2.1 初步

输入图像$I \in R^{W\times H \times 3}$，目标是产生一个关键点heatmap$\hat{Y}\in[0,1]^{\frac{W}{R}\times \frac{H}{R} \times C}$，$R$为输入步长，$C$是关键点类型的数量。姿态估计中$C=17$，目标检测中$C=80$。$\hat{Y}_{x,y,c}=1$对应检测的关键点，$\hat{Y}_{x,y,c}=0$则为背景。作者使用几个不同的全卷积编码解码网络从$I$来预测$\hat{Y}$:**hourglass,ResNet,DLA。**

对于每个类别$c$的GT关键点$p$，计算它的低分辨率$\overline{p}=\lfloor \frac{p}{R} \rfloor$，然后将GT所有点的映射到hearmap中$Y=\in[0,1]^{\frac{W}{R}\times\frac{h}{R}\times c}$。作者通过使用高斯核$Y_{xyc}=exp(*)$把所有gt关键点映射到heatmap $Y\in[0,1]^{\frac{W}{R}\times \frac{W}{R} \times C}$，如果同一个类的高斯是重叠的，则选择像素最大那个。

训练目标是采用的Focal Loss的像素回归:

![](https://gitee.com/weifagan/MyPic/raw/master/img/CenterNet loss.png)

为了弥补由输出步长导致的离散化误差，额外预测局部偏移。所有类别$c$共享一个偏移预测。

![](https://gitee.com/weifagan/MyPic/raw/master/img/CenterNet loss off.png)

### 2.2 把目标作为一个点

$(x^{(k)}_1,y^{(k)}_1,x^{(k)}_2,y^{(k)}_2)$为类别$c_k$的对象$k$的边框，它的中心点位于$(\frac{x^{(k)}_1+y^{(k)}_1}{2},(\frac{x^{(k)}_1+y^{(k)}_1}{2})$。作者用关键点估计器$\hat{Y}$来预测所有中心点。此外，还回归每个对象$k$的大小$s_k$。为了限制计算负担，对所有目标仅仅使用一个单尺寸预测$\hat{S}$。

![](https://gitee.com/weifagan/MyPic/raw/master/img/CenterNet loss size.png)

总的目标函数：

![](https://gitee.com/weifagan/MyPic/raw/master/img/CenterNet loss det.png)

对于每个位置，网络预测一共$C+4$输出(关键点+offset+size)。所有输出共享一个共同的全卷积backbone网络。对于每个模态，主干的特征通过3×3卷积、ReLU卷积和另一个1×1卷积。

**From points to bounding boxes**:推理时候，首先单独地给每个类别提取heatmap的峰值，作者检测所有大于等于它周围8个邻域的值响应并保持top100个峰值。作者使用关键点的值$\hat{Y}_{x_iy_ic}$作为它的检测自信度并产生边框：

![](https://gitee.com/weifagan/MyPic/raw/master/img/CenterNet box.png)

所有输出直接从关键点估计产生，不需要NMS或者其他后处理。峰值关键点提取看作是充足NMS替代方案并可以使用一个3x3的maxpool操作有效实现。



## 3 实验结果

![](https://gitee.com/weifagan/MyPic/raw/master/img/ap12.PNG)

CenterNet with Hourglass-104 achieves an AP of 45:1%, outperforming all existing
one-stage detectors。

## 4 总结

CenterNet把对象看做一个中心点。首先在全卷积网络上产生一个heatmap，然后heatmap上会有预测出来的中心点(峰值)，其他性能如大小维度等也可以从中心位置的图像特征进行回归。当然原图GT的坐标也要通过步长和高斯核映射到heatmap，这样子才能产生loss。



## 5 指导意义

* 把对象建模成一个中心点并在目标检测转换为关键点估计
* centernet网络

# Receptive Field Block Net for Accurate and Fast Object Detection
## 1.论文主要解决什么问题？

现有的主流目标检测器的套路：

1.依靠强有力的主干网络来提取特征(ResNet-101、Inception)，但是开销大，速度慢。

2.使用轻量级的主干网络(如mobilenet)，虽然速度快，但是精度较弱。

作者受到人类视觉系统感觉结构的启发，考虑感受野的大小和偏心率，提出了RFB模块来增加特征判别性和鲁棒性。

## 2. 所提算法如何解决问题？

## 2.1 



# Focal Loss for Dense Object Detection

## 1.论文要解决什么问题？

two-stage-detector可以达到很高的准确度，但是速度慢，two-stage-detector速度快，但是准确度较低，作者提出focal loss是想one-stage-detector也能达到two-stage-detector的准确度。



## 2.所提算法如何解决问题？

作者认为one-stage-detector的准确度不如two-stage-detector是由于样本的类别不均衡导致的。目标检测，一张图片可以有很多候选框，但是只有很少部分包含检测对象，这会带来类别不平衡。这种负样本太多导致占loss的大部分，而且多是容易分类的。

OHEM的思想虽然也是处理样本不平衡的问题，但是忽略了容易分类的样本。

因此针对类别不均衡问题，作者提出一种新的损失函数：focal loss，这个损失函数是在标准交叉熵损失基础上修改得到的。**这个函数可以通过减少易分类样本的权重，使得模型在训练时更专注于难分类的样本。**为了证明focal loss的有效性，作者设计了一个dense detector：RetinaNet，并且在训练时采用focal loss训练。**实验证明RetinaNet不仅可以达到one-stage detector的速度，也能有two-stage detector的准确率。**



## 2.1交叉熵

先看二分类的交叉熵。

![](https://gitee.com/weifagan/MyPic/raw/master/img/focalloss.jpg)

其中p表示正样本的预测概率。如果p=0.9，那么-log(p)固然也很小。

用pt替代p

![](https://gitee.com/weifagan/MyPic/raw/master/img/focalloss1.jpg)

那么公式(1)可以写成

![](https://gitee.com/weifagan/MyPic/raw/master/img/focalloss.PNG)

## 2.2 交叉熵改进

目标检测训练时候正负样本差距比较大，通常给正负样本加上权重，因为负样本数量多，所以给予的权重小一些，正样本数量小，所以给予的权重大一些。因此可以通过$\alpha_t$的值来调整正负样本对loss的共享权重。

![](https://gitee.com/weifagan/MyPic/raw/master/img/focalloss3.PNG)

虽然公式(3)可以控制正负样本的权重，但是没办法控制容易分类和难分类的权重。于是有了focal loss

![](https://gitee.com/weifagan/MyPic/raw/master/img/focalloss4.PNG)

$\gamma$称为focusing parameter，${(1-p_t)}^{\gamma}$称为调制系数，目的在于减少易分类样本的权重，使得模型更加专注于难分类样本的样本。

**Focal Loss 两个重要性质**

1.当一个正样本被分错类时，pt很小，那么调制因子(1-pt)接近1，损失不太收到影响，当pt接近1时候，1-pt很小，就是容易分类的样本的权重被给予很小的值。

2.当$\gamma=0$的时候，focal loss就是传统的交叉熵，$\gamma$能够使得难分类样本的权重，增加那些误分类的重要性。例如，当$\gamma$为2时，pt=0.9的loss要比标准的交叉熵小100+倍，当pt=0.968时，要小1000+陪。对于hard example 权重相对提升很多。

结合公式(3)，得到公式(5)，这样子既能调整正负样本的权重，又能控制难易分类样本的权重。

![](https://gitee.com/weifagan/MyPic/raw/master/img/focalloss6.PNG)

为了验证focal loss，作者提出RetinaNet(本质上就是resnet+fpn+fcn)，使用两个子网络来分别预测classes和box。

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/RetinaNet.PNG" style="zoom:80%;" />

## 3.实验结果如何

$\gamma$=2，$\alpha$=0.25时，效果最好，结果SOTA.

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/focallloss_res.PNG" style="zoom:80%;" />

## 4.思考

1.为什么二阶段的检测器可以避免类别不平衡问题？

因为二阶段检测器有RPN罩着。第一阶段的RPN对anchor进行简单分类(区分前景还是背景)，经过筛选，属于背景的bbx被大量削减，虽然其数量依然远大于前景类bbox，但是至少数量差距已经不像最初生成的anchor那样夸张了。就等于是 从 “**类别 极 不平衡**” 变成了 “**类别 较 不平衡**”

2.为什么一阶段检测器难以避免类别不平衡的问题？

因为one stage系的detector直接在首波生成的“类别极不平衡”的bbox中就进行难度极大的细分类，意图直接输出bbox和标签（分类结果）。而原有交叉熵作为分类任务的损失函数，无法抗衡“类别极不平衡”，容易导致分类器训练失败。因此，one-stage detector虽然保住了检测速度，却丧失了检测精度。

3.正负样本和难易分类样本有什么关系？

首先样本的不平衡可能会存在于正负样本，难易样本，类别间样本。Focal loss主要是解决难易样本不平衡的问题。

* 正样本：标签区域内的图像区域。

* 负样本：标签以外的图像区域，即图像背景区域。
* 易分类样本(容易识别正确的样本)：样本数量多，累计loss大，那么在训练过程中，loss的降低很大程度来源于这种数量多的样本，使得模型更加专注于学习这种样本，导致了它是易分类的样本。那么负样本通常来说是易分类的样本，那存在负样本也是难分类样本吗？就是这种负样本很容易识别成正样本是否存在？例如？可能存在背景跟前景很相似，使得它是难分类样本。譬如说背景有一个假人，那么这个负样本可能就是难分类样本。

* 难分类样本(难以识别正确的样本)：样本数量小，累计loss小，使得模型难以学习这种样本。

# SWA Object Detection

## 1.解决什么问题？

作者提出了一个有用的trick，不用增加推理耗时也不用改变检测器，就可以在COCO上提升1%AP。**方法是：使用周期学习率来额外训练检测器12个epoch，然后平均这些checkpoints作为最后的模型。**

作者提出的trick是受到SWA( Stochastic Weights Averaging，一种提高泛化能力的方法)的启发，发现SWA用在目标检测非常有用。

## 2.Trick为何有用？

### 2.1 SWA

简单来说，SWA就是在带有高恒定学习率或者周期性学习率的SGD的优化轨迹上平均多个checkpoints的方法，提升泛化能力。为啥简单的SWA有用？记第$i$个epoch的checkpoint为$w_i$，一般情况下，我们会使用最后epoch的模型$w_n$或者在验证集上效果最好的模型$w_i^*$作为最终的模型。但是SWA是平均多个模型$w=\frac{1}{n-m+1}\sum_{i=m}^n{w_i}$作为最终的模型。

SWA的具体做法如下图所示，前75%的时间使用标准的衰减学习速率策略训练，然后剩余25%设置一个合理的固定学习速率进行训练，最后平均第二阶段每个epoch的weights。如下图b所示，也可以采用在每个epoch采用周期式的学习速率策略来训练。另外一点是模型中如果有BN层，那么应该用SWA得到的模型在训练数据中跑一遍得到BN层的running statistics。

![](https://gitee.com/weifagan/MyPic/raw/master/img/swa2.PNG)

那么SWA为什么有效呢，论文也给了简单的解释，由于模型的参数属于高维空间，SGD训练的模型往往收敛到最优解的边界区域，如下图a中的模型![[公式]](https://www.zhihu.com/equation?tex=w_1), ![[公式]](https://www.zhihu.com/equation?tex=w_2)和![[公式]](https://www.zhihu.com/equation?tex=w_3)都落在边缘位置，但是**平均它们可以接近最优解**。那么SWA后面采用固定学习速率或者周期式学习速率来寻找更多的次优解，最后平均接近最优解。图b和c是说的是训练误差和测试误差往往不对齐，就是我们所说的模型泛化性能，那么平均的话其实是可以提升泛化性能的。

![](https://gitee.com/weifagan/MyPic/raw/master/img/swa4.PNG)

### 2.2 SWA应用在目标检测

但是SWA该如何应用到目标检测呢？

第一，SWA用什么学习率策略？使用高的固定的学习率还是周期性学习率？

第二，应该平均多少个checkpoints?

于是，作者开始做实验...

从实验结果来看，采用固定学习速率最终的模型效果有所恶化，但是采用cos学习速率效果有提升，具体地采用cos lr为(0.02, 0.0002)，额外训练12个epoch就可以额外提升约一个点。另外这个策略也在Faster R-CNN，RetinaNet，FCOS，YOLOv3和VFNet实验，最终都可以大约提升AP一个点左右。所以最后的策略是：

![](https://gitee.com/weifagan/MyPic/raw/master/img/swa6.PNG)

其中cyclical learning rates 如下图：

![](https://gitee.com/weifagan/MyPic/raw/master/img/swa1.PNG)

## 3.指导意义？

1.把SWA应用到目标检测上