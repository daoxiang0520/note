---
title: AI相关
date: 2026-06-15
source: 飞书文档 (Hm3qwcMj8i3Z6gkNhWycx4vonBc)
tags:
  - AI
  - 深度学习
  - 机器学习
---

# Numpy相关

## 函数参数

axis=i:沿i维操作

keepdim=True:保留原来维度，但是操作的维度大小缩减为1

## 向量操作

**数组索引:**

![](Attachments/AI/img_001.png)

![](Attachments/AI/img_002.png)

![](Attachments/AI/img_003.png)

![](Attachments/AI/img_004.png)

E.g.

dscore[range(dscore.shape[0]),y]-=1

对所有i in range,第i行第y[i]列的元素-1

# 梯度下降

提要:

损失函数:Softmax

$Loss=\sum_{i=1}^{N}-log(\frac{e^{z_{y_i}}}{\sum_{j}e^{z_j}})$(N类)

## 随机梯度下降(SGD)

```Python
while True:
    data_sample=sample(data,256)
    dW=gradient(W,data_sample)
    W=W-lr*dW
```

## 批量梯度下降

对全部样本迭代

速度极慢

```Python
while True:
    dW=gradient(W,data)
    W=W-lr*dW
```

## 优化

### 正则化

奥卡姆剃刀原理，将权重矩阵最小化，防止过拟合

则对损失函数改进为

$$Loss'=Loss+\lambda R(W)$$

#### L1正则化

利用绝对值

$$R_{L_1}(W)=\sum_j|W_j|$$

此时将W限制在以原点为中心的方形块内

使小权重趋向0

#### L2正则化

利用平方

$$R_{L_1}(W)=\sum_j W_j^2$$

此时将W限制在以原点为中心的球形区域内

防止某个权重过大以拟合数据集

### Momentum(动量优化)

在梯度下降时加入动量概念，使梯度下降保持连贯的速度

考虑有

$$v=\rho v+\nabla f(x) \\\rho 为阻力系数，通常接近1$$

则迭代方程变为

$$W=W-lr*v$$

```Python
v=0
decay_rate=0.9
while True:
    v=decay_rate*v-dW
    W-=lr*v
```

### RMSProp(均方根传播)

改变梯度下降幅度，下降幅度大是使步长变小，反之同理

考虑有

$$square=\alpha sqaure+(1-\alpha)dW^2\\W=W-\frac{lr*dW}{\sqrt{square}+1e-8}$$

```Python
square=0
decay_rate=0.9
while True:
    square=decay_rate*square+(1-decay_rate)*dW*dW
    W=W-lr*dW/numpy.sqrt(square+ le-8)
```

动态控制步长

### Adam

Adam将Momentum与RMSProp结合起来

有

![](Attachments/AI/img_005.png)

考虑第一步时动量参数与均方参数均为0，无法快速适应，加入偏差纠正

### AdamW

在Adam的基础上加入正则化

![](Attachments/AI/img_006.png)

注意正则化不在损失函数进行

而在迭代时加入正则化

## 损失计算/反向传播实例

### Backward Pass

对于一个前向公式 $y=Wx+b$

其中$x$shape为(N,D),N为样本数，D为特征数

那么在反向传播过程中对于公共参数(即b)将反向传回的梯度在样本维度相加，保留特征维度

对于独有参数（即x和W）则对每个元素都保留即 $dout=dx*W,dW=x^T*dout$

注意求导后shape不变

#### General Situation

对于一般的前向传播 $y=Wx+b$

已知shape $y->(N,M)\\x->(N,D)\\W->(D,M)$

那么 $\frac{\partial L}{\partial W}=\frac{\partial L}{\partial y}\frac{\partial y}{\partial W}\\\frac{\partial y}{\partial W}=x\\则\frac{\partial L}{\partial W}=x^T*dout$

其他情况类似

#### Batch Normalization

批归一化公式有 $y=\gamma \hat{x}+\beta\\\hat{x}=\frac{x-\mu}{\sqrt{\sigma^2+\epsilon}}$

与一般情况不同的是， $\gamma与\beta$为公共参数，形状为(D,),计算时通过广播机制作用到每一个样本

那么反向传播时，公共参数则表现为所有样本之和，包括一般情况下的 bias矩阵也一样

所以有 $\frac{\partial L}{\partial \gamma}=\sum_N dout*\hat{x}$（dout与x均为(N,D),此处为逐元素相乘）

当然 $\frac{\partial L}{\partial \beta}=\sum_N dout$

特别的，对于x

$$\frac{\partial y}{\partial x_i} = \frac{\partial y}{\partial \hat{x}_i} \frac{\partial \hat{x}_i}{\partial x_i} + \sum_{j=1}^{N} \left( \frac{\partial y}{\partial \hat{x}_j} \frac{\partial \hat{x}_j}{\partial \mu} \frac{\partial \mu}{\partial x_i} \right) + \sum_{j=1}^{N} \left( \frac{\partial y}{\partial \hat{x}_j} \frac{\partial \hat{x}_j}{\partial \sigma^2} \frac{\partial \sigma^2}{\partial x_i} \right)$$

一一对应后有

$$\frac{\partial y}{\partial x_i} = \gamma*\frac{1}{\sqrt{\sigma^2+\epsilon}} + \sum_{j=1}^{N} \left( \gamma*\frac{-1}{\sqrt{\sigma^2+\epsilon}}*\frac{1}{N} \right) + \sum_{j=1}^{N} \left( \gamma*((-0.5)(x_j-\mu)(\sigma^2+\epsilon)^{-1.5})*\frac{2(x_i-\mu)}{N} \right)\\=\frac{\gamma}{\sqrt{\sigma^2+\epsilon}}(1-1-\sum_j\frac{x_j-\mu}{N(\sigma^2+\epsilon)})\\=0$$

但注意y是x归一化形式，该结果意味着平移与拉伸不改变归一化结果，因此必须使用L来推导

$$\frac{\partial L}{\partial x_i} = \frac{\partial L}{\partial \hat{x}_i} \frac{\partial \hat{x}_i}{\partial x_i} + \sum_{j=1}^{N} \left( \frac{\partial L}{\partial \hat{x}_j} \frac{\partial \hat{x}_j}{\partial \mu} \frac{\partial \mu}{\partial x_i} \right) + \sum_{j=1}^{N} \left( \frac{\partial L}{\partial \hat{x}_j} \frac{\partial \hat{x}_j}{\partial \sigma^2} \frac{\partial \sigma^2}{\partial x_i} \right)$$

其中$\frac{\partial L}{\partial \hat{x_i}}=\frac{\partial L}{\partial y}\frac{\partial y}{\partial \hat{x_i}}$

$$\frac{\partial L}{\partial x_i} = \gamma*\frac{1}{\sqrt{\sigma^2+\epsilon}}*dout_i + \sum_{j=1}^{N} \left( \gamma*\frac{-1}{\sqrt{\sigma^2+\epsilon}}*\frac{1}{N}*dout_j \right) + \sum_{j=1}^{N} \left( \gamma*((-0.5)(x_j-\mu)(\sigma^2+\epsilon)^{-1.5})*\frac{2(x_i-\mu)}{N}*dout_j \right)\\=\frac{\gamma}{\sqrt{\sigma^2+\epsilon}}(dout_i-\sum_j\frac{dout_j}{N}-\sum_j\frac{\hat{x_j}(x_i-\mu)dout_j}{N\sqrt{\sigma^2+\epsilon}})\\=\frac{\gamma}{\sqrt{\sigma^2+\epsilon}}(dout_i-\sum_j\frac{dout_j}{N}-\sum_j\frac{\hat{x_j}\hat{x_i}dout_j}{N})$$

注意这是第i样本单特征公式

那么向量形式为

$$dx=\frac{\gamma}{N\sqrt{\sigma^2+\epsilon}}(Ndout-\sum_Ndout-\sum_Ndout*\hat{x})$$

由上面公式可知尾项乘积为逐项相乘

### Softmax loss

$$Loss=\sum_{i=1}^{N}-log(\frac{e^{z_{y_i}}}{\sum_{j}e^{z_j}})+\lambda \sum_j W_j^2$$

$$\frac{dLoss}{dz_i}=P(z_i)(i\ne y_i)$$

$$\frac{dLoss}{dz_y}=P(z_y)-1$$

$$后补正则化求导2\lambda W$$

```Python
def softmax_loss_vectorized(W, X, y, reg):
    """
    Softmax loss function, vectorized version.

    Inputs and outputs are the same as softmax_loss_naive.
    """
    # Initialize the loss and gradient to zero.
    loss = 0.0
    dW = np.zeros_like(W)
    scores=X.dot(W)
    scores-=np.max(scores,axis=1,keepdims=True)
    p = np.exp(scores)
    p /= p.sum(axis=1,keepdims=True)
    logp=np.log(p)
    dscore=p.copy()
    dscore[range(dscore.shape[0]),y]-=1
    dW=np.dot(X.T,dscore)
    loss-=np.sum(logp[range(logp.shape[0]),y])
    num_train=X.shape[0]
    loss = loss / num_train + reg * np.sum(W * W)
    dW /= num_train
    dW+=2*reg*W

    return loss, dW
```

### Hinge loss(SVM loss)

![](Attachments/AI/img_007.png)

```Python
# 1. 计算所有样本的 margin (N, C)
scores = X.dot(W)
correct_class_scores = scores[np.arange(num_train), y].reshape(-1, 1)
margins = np.maximum(0, scores - correct_class_scores + 1)
margins[np.arange(num_train), y] = 0 # 正确类不计入

# 2. 建立 Mask 矩阵 (N, C)
# 只要 margin > 0 的地方，标记为 1
mask = np.zeros(margins.shape)
mask[margins > 0] = 1

# 3. 处理正确类别的梯度
# 每一行 1 的总和，就是该样本违规的次数
incorrect_counts = np.sum(mask, axis=1)
mask[np.arange(num_train), y] = -incorrect_counts

# 4. 一次性算出权重梯度 dW (D, C)
# (D, N) dot (N, C) -> (D, C)
dW = X.T.dot(mask)

# 5. 最后处理：除以样本数，加上正则化项的导数
dW /= num_train
dW += 2 * reg * W
```

# 神经网络

## 预处理与优化

### Normalization

#### Batch normalization

https://arxiv.org/pdf/1502.03167

![](Attachments/AI/img_008.png)

将同一样本所有数据归一化

对CNN来说，即将同一Channel的数据归一化

#### Layer normaliization

将同一样本所有特征归一化

对CNN来说，即将同一位置的所有Channel数据归一化

#### 处理公式

Forwards:

![](Attachments/AI/img_009.png)

Backwards:

详见损失计算/反向传播实例

关于归一化与其公式：https://arxiv.org/pdf/1502.03167

$$\frac{\partial l}{\partial x_i} = \frac{\partial l}{\partial \hat{x}_i} \frac{\partial \hat{x}_i}{\partial x_i} + \sum_{j=1}^{N} \left( \frac{\partial l}{\partial \hat{x}_j} \frac{\partial \hat{x}_j}{\partial \mu} \frac{\partial \mu}{\partial x_i} \right) + \sum_{j=1}^{N} \left( \frac{\partial l}{\partial \hat{x}_j} \frac{\partial \hat{x}_j}{\partial \sigma^2} \frac{\partial \sigma^2}{\partial x_i} \right)$$

![](Attachments/AI/img_010.png)

### Dropout

防止过拟合，让模型真正学习特征

![](Attachments/AI/img_011.png)

Parameter：Probability of dropout

![](Attachments/AI/img_012.png)

Train：Dropout

Test：必须将测试时神经元信号强度与训练时平衡

训练时仅有p的信号强度（p的概率dropout神经元）

测试时使用全部神经元——>输出信号\*p来平衡强度

![](Attachments/AI/img_013.png)

![](Attachments/AI/img_014.png)

实现：mask掩码筛选

## Activation Function(激活函数)

激活函数——引入非线性

### Sigmoid

$$\sigma(x)=\frac{1}{1+e^{-x}}$$

Problem:Vanishing Gradient（梯度消失）

![](Attachments/AI/img_015.png)

如图，导数在极端情况非常小，导致反向传播时的梯度消失

### ReLU

$$f(x)=max(x,0)$$

良好的梯度保持

Problem:x<0时的神经元死亡

![](Attachments/AI/img_016.png)

### Leaky ReLU

$$f(x)=max(0.1x,x)$$

![](Attachments/AI/img_017.png)

### ELU

$$f(x)=\begin{cases}x & x\geq 0 \\\alpha (e^x -1) & x<0\end{cases}$$

![](Attachments/AI/img_018.png)

### GELU(Gaussian Error Linear Unit)

$$f(x)=x*\phi(x)$$

![](Attachments/AI/img_019.png)

Smoother

Problem:

1.Higher computational cost

2.Large negative values still have gradient->0

![](Attachments/AI/img_020.png)

### SiLU

$$f(x)=x*\sigma(x)\\\sigma(x)为Sigmoid函数$$

近似于GELU

## ResNet(残差网络)

Base：随网络深度增加，训练效果并不是一起提升，往往20层效果会优于50层（deeper models are harder to optimize）

1.Gradient Vanishing（梯度消失）

链式法则连乘，若一层贡献一直小于1，则通过深层叠加梯度趋近于0

2.Degradation（退化问题）

![](Attachments/AI/img_021.png)

3.Complex Loss Landscape（误差平面太复杂）

深层神经网络的误差平面太复杂，优化器易于陷入非常多的局部最优

Solution：Residual connection（残差连接）

![](Attachments/AI/img_022.png)

梯度保底为1，即什么都不学原样传递，那么从底层神经元到顶层神经元可以无损传输原始数据

变换后形状不同？1x1卷积，通过改变步长来变换形状

## Weight Initialization

![](Attachments/AI/img_023.png)

### Kaiming Initialization

由于ReLU对初始权重矩阵会去掉一半的权重,为了保持信号强度与方差，可以将信号强度翻倍来补偿

![](Attachments/AI/img_024.png)

![](Attachments/AI/img_025.png)

## CNN

![](Attachments/AI/img_026.png)

![](Attachments/AI/img_027.png)

### Basic Principal

![](Attachments/AI/img_028.png)

### Paddings

![](Attachments/AI/img_029.png)

问题：在迭代过程中特征图不断缩小，带来边缘信息丢失与网络深度限制

![](Attachments/AI/img_030.png)

![](Attachments/AI/img_031.png)

注意：Padding仅在卷积层中处理，连接层与池化层不处理padding（Scale无损失无影响）

![](Attachments/AI/img_032.png)

### Receptive Fields(感受野)

![](Attachments/AI/img_033.png)

最终输出的像素感受野有限，我们希望聚合整个图的信息——>改变Stride（步幅）——>有效感受野增加

![](Attachments/AI/img_034.png)

### Pooling Layer(池化层)

无需Padding

![](Attachments/AI/img_035.png)

有max，average等

### Batch normalization

![](Attachments/AI/img_036.png)

### Architechure

![](Attachments/AI/img_037.png)


![](Attachments/AI/img_038.png)

常见架构

### Data processing

![](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDQ3ZGU5MmU1NzhlYWU4ZmZjNzllOTc2YzQ0YzI1MTZfYjlmZmUyMzcwNWFmMmVkYzBjNmZlMGY4YzA1OTBkYjlfSUQ6NzY0MDY4OTY3Mzg2OTg1NTcwN18xNzgxNTAxMDE1OjE3ODE1MDQ2MTVfVjM)

### Data augmentation(数据增强)

![](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YmVjNWM0ODY2NDM1OWU1ZjM0MTAzN2VmOWViZjU3M2FfYzY4YjZlNDljNWJmMzVlMzNlZTU2NjRkMDlmZmYxMjFfSUQ6NzY0MDY5MDEyOTA4MTgzMDM1Nl8xNzgxNTAxMDE0OjE3ODE1MDQ2MTRfVjM)

对数据/图层进行转换：水平翻转，随机调整图片crops/scales（裁切图片），测试时间增强，Color Jitter，Cutout

![](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=M2YxMTJmYzk2ZjBjODljYjQ3YmY3MzczOTVkZjM2ZTBfNDc4Y2Q0YTM3ZWY1YTZiODAxNDZjZDkyMWI2MGVmZDJfSUQ6NzY0MDY5MjkwMTI4NjMwMDg1M18xNzgxNTAxMDE1OjE3ODE1MDQ2MTVfVjM)

### Transfer learning

微调layer，冻结其他层来获得更好的结果

![](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTA3NWQ1NDhmZThhNGZkNGRkNWIwMTQ5ZDVmYTY5MjVfNGVlNjlkZjU0MDc4NWE5NzYxODA0NmQ4ZTY0NGM1ZDNfSUQ6NzY0MDY5NTA5NDk3ODQxNTgxNF8xNzgxNTAxMDE0OjE3ODE1MDQ2MTRfVjM)

### Hyperparameters

Learning rate:1e-1~1e-5

![](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZTllZDg1ODk4MjAwOTQ4YjcwZTMyMTdkZTI4Njg3MTlfMWUzODgzMTEwMjQ4ZGM3YzNmMzhkN2JlNGQxZjdkYWRfSUQ6NzY0MDY5NjQyODEwODQ4MzUzNF8xNzgxNTAxMDE1OjE3ODE1MDQ2MTVfVjM)
