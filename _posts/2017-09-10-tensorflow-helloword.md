---
title: Tensorflow学习
key: 20170910
tags: tensorflow 'machine learn' 
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---

google tensorflow初接触
<!--more-->
### 一、环境搭建 

#### 1.安装Anaconda

>  Anaconda 是一个用于科学计算的 Python发行版，让我们可以方便的进行包管理与环境管理。

**下载安装**

​	Anaconda 安装包可以到 https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/ 下载。

​	TUNA 还提供了 Anaconda 仓库的镜像，运行以下命令,即可添加 Anaconda Python 免费仓库。

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/ conda config
--set show_channel_urls yes
```

 **创建python3.5运行环境**
​	Anaconda3默认安装的是Python3.6，但是目前tensorflow只支持python3.5，但是目前的tensorflow只支持python3.5，这个时候要重新创建一个python3.5的环境，具体步骤：

- 创建一个名为python3.5的环境，命名为python35

  打开Anaconda Prompt,输入 conda create --name python35 python=3.5

- 安装完成后，激活和取消激活命令

  激活：activate python35

  取消激活：deactivate

这个时候可以在Anaconda3路径下envs下看到刚才创建的python35，每当我们激活python3.5时，系统的运行环境就在这个文件下面。

#### 安装Tensorflow

安装好了上述的python3.5的环境之后，先激活python35，我们这里要装的是gpu版

输入：pip install tensorflow-gpu
若pip下载速度太慢，设置下pip镜像源[^1]

完成后，虽然安装完成了，但是需要GPU加速，还需要安装cuda和cuDnn（专门为deep learning 准备的加速库）

#### 安装CUDA Toolkit 9.0
  1. [下载安装exe文件](https://developer.nvidia.com/cuda-toolkit-archive)
    默认安装即可，会自动在环境变量添加
  2. 验证CUDA是否安装成功 

![查看cuda信息][1]

  3. 验证用户环境变量
     CUDA_PATH已经加到环境变量中了，然后把bin和lib/x64加到环境变量中
     ![enter description here][2]

#### CuDnn库下载

1. [下载zip文件](https://developer.nvidia.com/rdp/cudnn-download)

  ![enter description here][3]

2. 解压文件到CUDA目录
    解压cudnn-9.0-windows10-x64-v7.zip，将解压的三个文件夹拷贝至CUDA目录，进行覆盖即可。默认文件夹在：  `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0` 

#### 验证
在tensorflow环境里打开python

```python
import tensorflow as tf
hello = tf.constant("Hello!TensorFlow")
sess = tf.Session()
print(sess.run(hello))
```
![gpu加速程序][4]



### 二、基本概念
#### 什么是Tensorflow
TensorFlow™ 是一个采用数据流图（data flow graphs），用于数值计算的开源软件库。


节点（Nodes）在图中表示数学操作，图中的线（edges）则表示在节点间相互联系的多维数据数组，即张量（tensor）。
#### 什么是数据流图
数据流图用“结点”（nodes）和“线”(edges)的有向图来描述数学计算。“节点” 一般用来表示施加的数学操作，但也可以表示数据输入（feed in）的起点/输出（push out）的终点，或者是读取/写入持久变量（persistent variable）的终点。“线”表示“节点”之间的输入/输出关系。这些数据“线”可以输运“size可动态调整”的多维数据数组，即“张量”（tensor）。张量从图中流过的直观图像是这个工具取名为“Tensorflow”的原因。一旦输入端的所有张量准备好，节点将被分配到各种计算设备完成异步并行地执行运算。

***HelloWorld***

``` python
import tensorflow as tf
import numpy as np

# 使用 NumPy 生成假数据(phony data), 总共 100 个点.
x_data = np.float32(np.random.rand(2, 100)) # 随机输入
y_data = np.dot([0.100, 0.200], x_data) + 0.300

# 构造一个线性模型
# 
b = tf.Variable(tf.zeros([1]))
W = tf.Variable(tf.random_uniform([1, 2], -1.0, 1.0))
y = tf.matmul(W, x_data) + b

# 最小化方差
loss = tf.reduce_mean(tf.square(y - y_data))
optimizer = tf.train.GradientDescentOptimizer(0.5)
train = optimizer.minimize(loss)

# 初始化变量
init = tf.initialize_all_variables()

# 启动图 (graph)
sess = tf.Session()
sess.run(init)

# 拟合平面
for step in xrange(0, 201):
    sess.run(train)
    if step % 20 == 0:
        print step, sess.run(W), sess.run(b)

# 得到最佳拟合结果 W: [[0.100  0.200]], b: [0.300]
```
### 三、Tensorflow入门
#### 1.计算模型-计算图
tensorflow中的所有计算都会转化为图中的节点

使用：
①默认计算图（默认已注册）
``` python
c = tf.constant(4.0)
assert c.graph is tf.get_default_graph()
```
②设置当前计算图为默认
``` python
#Graph上下文管理器，在上下文的生命周期中覆盖当前的默认图形
# 1. Using Graph.as_default():
g = tf.Graph()
with g.as_default():
  c = tf.constant(5.0)
  assert c.graph is g

# 2. Constructing and making default:
with tf.Graph().as_default() as g:
  c = tf.constant(5.0)
  assert c.graph is g
```
计算图可以通过tf.Graph.device函数 指定运行计算的设备
```python
#创建计算图
g=tf.Graph()
with g.device('/gpu:0'):
  a = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]], name='a')
  b = tf.constant([[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]], name='b')
  c = tf.matmul(a, b)
#通过ConfigProto protocol Buffer来配置需要生成的会话    log_device_placement：True gpu运算不满足时自动切cpu运算
config=tf.ConfigProto(log_device_placement=True)
#创建会话
sess = tf.Session(config=config)
print(sess.run(c))
 #[[22. 28.]
 #[49. 64.]]
```



#### 2.数据模型-张量
可以简单理解为多维数组，但是在tensorflow中的实现并不是直接采用数组的形式，它只是对tensorflow中运算结果的引用，在张量中并没有真正保存数字，保存的是如何得到这些数字的计算过程

使用：
``` python
import tensorflow as tf
a=tf.constant([1.0,2.0],name='a')
b=tf.constant([2.0,3.0],name='b')
result=tf.add(a,b,name='addOp')
print(result)
Tensor("addOp:0", shape=(2,), dtype=float32)
#名称  维度  类型
```
①对中间结果的引用,如上面的a节点 
②在session会话中，张量可以用来获取计算结果

#### 3.运行模型-会话
tensorflow中用会话来执行**定义好的运算**

使用：
``` python
#创建一个会话，并通过python中的上下文管理器来管理这个会话
with tf.Sessin() as sess:
    #使用这创建好的绘画来计算关心的结果
    sess.run(...)
#不需要再调用 “sess.close()”函数来关闭会话了
#当上下文退出时会话关闭和资源释放也自动完成
```

tensorflow会自动生成一个默认的计算图，如果没特殊的指定，计算会自动加入这个计算图中
但tensorflow中不会自动生成默认的会话，而是需要手动指定，当默认的会话被指定之后，可以通过tf.Tensor.eval函数来计算一个张量的取值
tensorflow提供了在交互环境下直接构建默认会话的函数 tf.InteractiveSession
``` python
sess=tf.Session()
with sess.as_default():#指定为默认会话
    print(result.eval())
#下面的两个命令有相同的功能
sess=tf.Session()
print(sess.run(result))
print(result.eval(session=sess))
#交互环境下直接构建默认会话
sess=tf.InteractiveSession()
print(result.eval())
sess.close()
```

#### Tensorflow游乐场
<http://playground.tensorflow.org>







[^1]: http://blog.csdn.net/ipaomi/article/details/78466321

  

  参考文献：
  https://www.cnblogs.com/elroye/p/7864988.html
  http://blog.csdn.net/lcb_coconut/article/details/79228759
  [TensorFlow学习笔记（一）入门][6]
  [TensorFlow 教程 - 深入MNIST完整代码](http://blog.csdn.net/toormi/article/details/53789562)
  [使用Tensorflow和MNIST识别自己手写的数字](http://blog.csdn.net/sparta_117/article/details/66965760)
  [TensorFlow_MNIST 保存、恢复模型及参数](http://blog.csdn.net/JerryZhang__/article/details/75535295)



  



[1]: assets/images/1520867629585.jpg
[2]: assets/images/1520867763992.jpg
[3]: https://i.loli.net/2018/03/13/5aa78d29cde7d.jpg
[4]: https://i.loli.net/2018/03/13/5aa79d428b6e7.jpg
[5]: http://www.tensorfly.cn/images/tensors_flowing.gif
[6]: http://blog.csdn.net/wuyzhen_csdn/article/details/64516733