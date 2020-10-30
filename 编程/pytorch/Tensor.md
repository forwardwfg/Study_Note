# Tensor 

# 导语

深度学习框架Pytorch发展势头惊人，这点小编也深有体会，翻翻Github上深度学习的开源代码，发现用Pytorch真的多了不少，所以小编最近也正在入坑Pytorch，顺便写写文章做些总结。认真看完这篇文章，你将收获：

* 理解Tensor的创建
* 理解Tensor的加速
* 理解Tensor的常用属性
* 理解Tensor的常用方法

## Tensor创建

我们应该都知道Numpy是支持大量维度数组与矩阵运算的常用扩展库。但是对于计算图，深度学习或者梯度，Numpy似乎真的有心无力，因为它的计算无法像Tensor一样在GPU上加速。今天我们就一起来谈谈Pytorch最基本的概念Tensor。

Tensor是n维的数组，在概念上与numpy数组是一样的，不同的是Tensor可以跟踪计算图和计算梯度。

1.从Numpy中创建

```python
import torch
import numpy as np

numpy_array= np.array([1,2,3])
torch_tensor1 = torch.from_numpy(numpy_array)
torch_tensor2 = torch.Tensor(numpy_array)
torch_tensor3 = torch.tensor(numpy_array)
```

值得注意的是torch.Tensor()是默认张量类型torch.FloatTensor()的别名，就是说返回torch.Tensor()返回的是Float数据类型。其实我们也可以修改其默认的数据类型：

```Python
torch.set_default_tensor_type(torch.DoubleTensor)
```

而torch.tensor()则会根据输入数据类型生成相应的torch.LongTensor、torch.FloatTensor和torch.DoubleTensor。

当然，我们也可以通过numpy()把Tensor转换为numpy类型

```python
numpy_array = torch_tensor1.numpy() # 如果tensor在CPU上
numpy_array = torch_tensor1.cpu.numpy() # 如果tensor在GPU上
print(type(numpy_array)) #输出 ： <class 'numpy.ndarray'>
```

> 注意，如果Tensor在GPU上，则需要使用.cpu()先将GPU的Tensor转换到cpu上。

> 注意，numpy()和from_nump()得到的数据与原数据是共享内存的，就像浅拷贝一样。
>
> 但是！！如果用torch.tensor()或者torch.Tensor()将numpy()转换为Tensor，就不共享内存了。可以理解成深拷贝把！

2.从Python内置类型中创建

```python
lst = [1,2,3]
torch_tensor1 = torch.tensor(a)
tp = (1,2,3)
torch_tensor2  = torch.tensor(a1)
```

3.其他方式

```python
# 创建相同元素的Tensor
torch_tensor1  = torch.full([2,3],2)
# 创建全为1的Tensor
torch_tensor2 = torch.ones([2,3])
# 创建全为0的Tensor
torch_tensor3 = torch.zeors([2,3])
# 创建对角阵的Tensor
torch_tensor4  = torch.eye(3)
# 在区间[1,10]中随机创建Tensor
torch_tensor5 = torch.randint(1,10,[2,2])
# 等等...
```

创建Tensor时候也可指定数据类型和所存储的设备

```python
torch_tensor= torch.zeros([2,3],dtype=torch.float64,device=torch.device('cuda:0'))
torch_tensor.dtype #torch.float64
torch_tensor.device #cuda:0
torch_tensor.is_cuda #True
```

## Tensor加速 

我们可以使用以下两种方式使得Tensor在GPU上加速。

第一种方式是定义cuda数据类型。

```python
dtype = torch.cuda.FloatTensor
gpu_tensor = torch.randn(1,2).type(dtype) #把Tensor转换为cuda数据类型
```

第二种方式是直接将Tensor放到GPU上(推荐)。

```PYTHON
gpu_tensor = torch.randn(1,2).cuda(0)#把Tensor直接放在第一个GPU上
gpu_tensor = torch.randn(1,2).cuda(1)#把Tensor直接放在第二个GPU上
```

而将Tensor放在CPU上也很简单。

```python
cpu_tensor = gpu_tensor.cpu()
```

## Tensor常用属性

1.查看Tensor类型属性

```python
tensor1 = torch.ones([2,3])
tensor1.dtype # torch.float32
```

2.查看Tensor尺寸属性

```python
tensor1.shape # 尺寸
tensor1.ndim #维度
tensor1.size()
```

3.查看Tensor是否存储在GPU上

```python
tensor1.is_cuda #False
```

4.查看Tensor存储设备

```python
tensor1.device # cpu
tensor1.cuda(0)
tensor1.device # cuda:0
```

5.查看Tensor梯度计算

```python
tensor1.grad
```

5.取单个值的tensor的值

```python
tensor2 = torch.tensor([2])
tensor1.item()
```



## Tensor常用方法

1.torch.squeeze()：删除值为1的维度并返回Tensor

```Python
tensor1 = torch.ones([2,1,3])
torch_tensor1.size() #torch.Size([2, 1, 3])
tensor2=torch.squeeze(tensor1)
print(tensor2.size())#torch.Size([2, 3])
```

从例子中可以看到Tensor的维度从原来的[2,1,3]变成[2,3],值为1的维度被删除。

2.torch.Tensor.permute()置换Tensor维度并返回新视图。

```python
tensor1 = torch.ones([2,1,3])
print(tensor1.size()) # torch.Size([2, 1, 3])
tensor2 = tensor1.permute(2,1,0) # 0,1,2-> 2,1,0
print(tensor2.size()) # torch.Size([3, 1, 2])
```

从例子中可以看到，Tensor将原本的第一个维度值为2，第2维度值为3，置换后分别变成3和2.

3.torch.Tensor.expand()：把值为1的维度进行扩展，扩展的Tensor不会分配新的内存，只是原来的基础上创建新的视图并返回。

```python 
>>>tensor1 = torch.tensor([[3],[2]])
>>>tensor2 = tensor1.expand(2,2)
>>>tensor1.size()
torch.Size([2, 1])
>>>tensor2
tensor([[3, 3],
        [2, 2]])
>>>tensor2.size()
torch.Size([2, 2])
```

从例子中可以看到，原本Tensor的维度是(2,1)，因为该函数是对值为1的维度进行扩展，所以可以扩展为(2,2),(2,3)等，但是要注意的是保持非1值的维度不变。

4.torch.Tensor.repeat()：沿着某个维度的Tensor进行重复，和expand()不同的是，该函数复制原本的数据。

```python
>>>tensor1 = torch.tensor([[3],[2]])
>>>tensor1.size()
torch.Size([2, 1])
>>>tensor2=tensor1.repeat(4,2)
>>>tensor2.size()
torch.Size([8, 2])
>>>tensor2
tensor([[3, 3],
        [2, 2],
        [3, 3],
        [2, 2],
        [3, 3],
        [2, 2],
        [3, 3],
        [2, 2]])
```

例子中的tensor1维度是(2,1)，tensor1.repeat(4,2)，这是对第0维度和第1维度的Tensor分别重复4次和2次，所以repeat后维度变成了(8,2)。再看下面的例子。

```python
>>>tensor1 = torch.tensor([[2,1]])
>>>tensor1.size()
torch.Size([1, 2])
>>>tensor2=tensor1.repeat(2,2,1)
>>>tensor2.size()
torch.Size([2,2,2])
>>>tensor2
tensor([[[2, 1],
         [2, 1]],

        [[2, 1],
         [2, 1]]])
```

例子中的tensor1维度是(1,2)，tensor1.repeat(2,2,1)，此时前者跟后者的维度对应不上，此时可以理解为tensor1把维度改写为(1,1,2)，再tensor1.repeat(2,2,1)，tensor1第0,1,2的维度的Tensor分别重复1次，1次，2次，repeat后Tensor维度为(2,2,2)。

5.torch.equal() and torch.eq() 前者是对整个Tensor进行比较，后者是对Tensor元素进行比较

```python
>>>tensor1 = torch.tensor([2,1])
>>>tensor2 = torch.tensor([2,2])
>>>torch.equal(tensor1,tensor2)
False
>>>torch.eq(tensor1,tensor2)
tensor([True,False])
```



## Tensor操作

1.算术操作

在pytorch中，容易操作有多种形式，以加法为例

* 加法形式1

```python
tensor1 = torch.tensor([2,1])
tensor2 = torch.tensor([2,1])
y = tensor1 + tensor
```

* 加法形式2

```python
y = torch.add(tensor1,tensor2
```

* 加法形式3 inplace

```python
tensor2.add_(tensor1)
```

>**注：PyTorch操作inplace版本都有后缀`_`, 例如`x.copy_(y), x.t_()`**

以上几种形式均输出

```python
tensor([4, 2])
```

2.索引

我们可以像Numpy一样通过索引来访问Tensor，但是要注意的是通过索引所得到的结果与原来的数据是共享内存的，就是说一个数据修改，另外一个数据也跟着修改。个人认为其实通过索引得到的结果就是浅拷贝

```python
tens1 = tensor1[1:] 
print(tensor1)
tens1 += 1
print(tensor1)
print(tens1)
print(id(tens1)==id(tensor1))
```

输出结果：

```python
tensor([2, 1])
tensor([2, 2])
tensor([2, 2])
False # 可以看到虽然共享内存，但是对象地址是不同的
```

但是，这样子

```python
tens1 = tensor1[1:] 
print(tensor1)
tens1 = ten1 + 1
print(tensor1)
print(id(tens1)==id(tensor1))
```

和这样子

```python
tens1 = tensor1[1:] 
print(tensor1)
tens1 = torch.add(ten1 ,1)
print(tensor1)
print(tens1)
print(id(tens1)==id(tensor1))
```

结果都是

```python
tensor([2, 1])
tensor([2, 1])
tensor([2, 2])
False
```

都没有改变原来数据，这是为什么呢？跟前面说得不一样啊？先留个疑问吧。大概应该是检查是否共享数据用+和torch.add()是没用的，因为他们都是创建新的对象和copy数据到新对象对应的内存单元。留个坑，写详细一点。

当然如果你想把结果指定到原地址，可以

```python
tens1 = tensor1[1:] 
tensor1[:] = ten1 + 1
print(id(tens1)==id(tensor1))
```

或者使用y.add_(x)

## 改变形状

Pytorch中可以通过view()来改变Tensor的形状，但是需要注意的是view()返回的Tensor与原来的数据是共享**data**的。view，顾名思义，是从不同的角度来观察数据。

```python
t = torch.rand(5,3)
t1 = t.view(3,5)
t2 = t.view(15)
t1 *= 2 
print(t1.size())
print(t2.size())
print(t)
print(t1)
```

输出结果：

```python
torch.Size([3, 5])
torch.Size([15])
torch.Size([3, 5])
torch.Size([15])
tensor([[0.6155, 1.4285, 0.5873],
        [1.7669, 0.5422, 0.3978],
        [0.2715, 1.3985, 0.2593],
        [0.9183, 1.0693, 0.1002],
        [0.5875, 1.3510, 1.7984]])
tensor([[0.6155, 1.4285, 0.5873, 1.7669, 0.5422],
        [0.3978, 0.2715, 1.3985, 0.2593, 0.9183],
        [1.0693, 0.1002, 0.5875, 1.3510, 1.7984]])
```

如果我们要得到新的副本，那可以先clone()得到新的副本再view()。其实改变形状还有reshape的方法，但是其不能保持能得到新的副本，所以推荐先clone一下数据。

```python
t = torch.rand(5,3)
t1 = t.clone().view(3,5)
t2 = t.clone().view(15)
t1 *= 2 
print(t1.size())
print(t2.size())
print(t)
print(t1)
```

数据结果：

```python
torch.Size([3, 5])
torch.Size([15])
tensor([[0.9997, 0.3064, 0.1957],
        [0.9084, 0.5462, 0.1061],
        [0.9929, 0.8168, 0.1814],
        [0.9040, 0.2284, 0.6171],
        [0.7444, 0.5135, 0.3031]])
tensor([[1.9993, 0.6128, 0.3915, 1.8168, 1.0923],
        [0.2123, 1.9858, 1.6336, 0.3627, 1.8081],
        [0.4568, 1.2341, 1.4888, 1.0270, 0.6062]])
```





公众号：CVpython**，专注于分享Python和计算机视觉，我们坚持原创，不定期更新，希望文章对你有帮助，快点扫码关注吧。

![](https://gitee.com/weifagan/MyPic/raw/master/img/QRcode_wechart.PNG)















