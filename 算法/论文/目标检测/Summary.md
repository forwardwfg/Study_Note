# 1 YOLO发展史

## 1.1 YOLOV1

### 1.1.1 YOLOV1的核心思想

* YOLOv1的核心思想是直接在输出层回归bounding box的位置长宽和类别。
* YOLOv1之前都是先提取候选框，然后把它送入到分类器就行分类是两个阶段的。而YOLOV1就是直接去回归bounding box的位置长宽类别。

### 1.1.2 YOLOV1的实现方法

* 将一幅图像分成sxs个网格，如果某个object落在这个网格中，那么这个网格负责预测这个object。

* 每个网格预测B个bounding box，每个boundingbox除了预测自身的位置之后，还要附带confidence。

  > confidece = p(object)*iou,没搞懂能计算，为什么还要预测？

* 损失函数都是误差平方求和，包括坐标，置信度(有没object都算上，其他并不是)和类别的损失计算。

  >1.8维度的坐标误差和20维的分类误差不对等，所以要取不同的权重，前者系数取5
  >
  >2.没有object的网格很多，系数取0.5
  >
  >3.有object的置信度和类别损失系数都取1
  >
  >4.预测处理的bounding box 的宽和长都取平方根，因为小的bounding box对偏差比较敏感

### 1.1.3 YOLOV1的其他细节

* 每个格子会预测2个bounding box，但是loss只计算与GT最大的bounding box的error，另外一个bounding box不做修正
* 标签中xy算的是格子中心点位置和object中心点的offset，而不是绝对的位置，但是预测的w,h的绝对值，不是谁的offset，x,y,w,h都是归一化到0-1的

### 1.1.4 YOLOV1的优缺点

* 速度比二阶段的模型很快

* YOLOv1对于相互靠近，还有很小的物体检测效果都不是很好，因为一个网格只预测了两个bouding box，而且是属于一个类别

* 同一个物体出现新的不常见的长宽比，泛化能力较弱

  

## 1.2 YOLOV2

相比于yolov1，精度和速度都有所提升。

### 1.2.1 Better

* BN:YOLO每个网络层加入BN，加快收敛

  > 加了BN就不用加dropout了

* 高分辨分类器：YOLOv2的分类网络以448x448的分辨率下在imagenet上做finetune。因为大多数分类器都是在imagenet预训练的，像素小于256x256，但是YOLO从224x224增加到448x448，所以finetune让模型适应大的分辨率

  > 这里的分类网络其实指的检测器的BackBone，因为backbone本来就是分类网络。

* Anchor：引入anchor机制，ap下降了，但是recall提升了7%。

* 聚类：YOLOv2是通过k-means来得到anchor，而YOLOv2之前的都是感觉经验来设置anchor的。距离度量使用d(box,centroid)=1-iou

* 绝对位置预测，公式并不是很懂，为什么更容易学习，更加稳定？

* 细粒度特征：在26x26的特征图经过卷积得到13x13，作者认为损失了很多细粒度特征，导致小物体检测效果欠佳，所以在此加入passthrough层，

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/passthrough.jpg" style="zoom:33%;" />

* 多尺寸训练:yolov2只有全卷积网络，对输入图片大小没有限制

### 1.2.1 Faster 

使用Darknet-19，该架构的网络参数较VGG-16更少，在ImageNet上，仍然可以达到top-1 72.9%以及top-5 91.2%的精度。

### 1.2.3 Stronger

提出了yolo 9000



## 1.3 YOLOv3

相比于yolov2，主要是精度提升了，但是它仍然很快，精度跟SSD差不多，但是速度是它的三倍。

改进之处:

* 类别预测：不再使用softmax，因为预测的bounding box可能会有重叠的类别(Softmax使得每个框分配一个类别)，所以用了logistic分类器来替代。分类损失使用二分类交叉熵
* 多尺度预测：FPN
* backbone：darknet-53



## 1.4 YOLOv4

相比于yolov3，ap和fps分别提升了10%和12%

**组成：**

* Backbone：CSPDarknet53
* Neck：SPP，PAN
* head：YOLOV3

**YOLO v4使用了：**

* BoF for backbone: CutMix和马赛克数据扩增，dropblock, class label smoothing
* BoF for backbone: Mish 激活函数，CSP，MiWRC
* BoF for detector: CIoU-loss，CmBN，DropBlock，马赛克，自对抗训练，Eliminate grid sensitivity, Using multiple anchors for a Cosine annealing scheduler, Optimal hyperparameters, Random training shapes
* BoS for detector: Mish activation,SPP-block, SAM-block, PAN path-aggregation block,DIoU-NMS

