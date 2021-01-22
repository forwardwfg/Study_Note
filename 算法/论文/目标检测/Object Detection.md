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