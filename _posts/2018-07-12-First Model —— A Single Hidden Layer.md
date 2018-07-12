---
layout: post
title: First Model —— A Single Hidden Layer
category: Machine Learning
permalink: /blog/Machine Learning/:title
---

[1.  赛题介绍及一些预备知识]({{baseurl}}/blog/{{page.category}}/Prerequisites-and-the-Data)

### tensorflow环境安装

在正式开始前想简单说一下tensorflow GPU版本的环境安装，这可不止`pip install tensorflow-gpu`这么简单。

首先最好建立一个jupyter的专用python环境，环境名字可以任取，关于创造python虚拟环境的教程请见[这里](https://www.cnblogs.com/technologylife/p/6635631.html)（记得把virtualenv增加到环境变量），关于如何在jupyter中增加刚刚创造的虚拟环境对应的kernel的方法请见[这篇教程](https://blog.csdn.net/Lazybones_3/article/details/78631232)（pip之前如何激活请参考之前的链接）。这里多说一句，虚拟环境下的pip方法很大概率会出问题，如果`pip install xxx`方法报错的话请尝试`python -m pip install xxx`。最后只要在jupyter界面的new菜单下见到对应的虚拟环境名称就成功啦~（如果你是先打开jupyter再尝试增加kernel的话记得在查看是否成功前刷新界面）。

![1531235367829](http://p4mu7u37x.bkt.clouddn.com/virtualenv.png)

再然后就是安装tensorflow了，tensorflow分为cpu版和gpu版，如果你没有符合要求的gpu（具体哪个版本以上可以我不太清楚，个人电脑是960m，可以借此作为参考），那么直接`pip install tensorflow`就可以了，之后就正常导入即可。但是如果你有符合要求的gpu，那么这时需要改成`pip install tensorflow-gpu`，而且还有一堆东西需要配置。具体的配置步骤见[这里](https://jingyan.baidu.com/article/d45ad14842d99969542b8054.html)，唯一过时的地方就是截止到tensorflow-gpu版本1.8以前，已经可以支持CUDA9.0，再然后注意对应的cuDNN即可。在上述所有这些环境都配置完后，导入tensorflow时依然可能显示错误信息`CUDA driver version is insufficient for CUDA runtime version`，这时请更新你的显卡驱动（建议直接下个GeForce Experience用来更新）。

到这里应该就能配置成功了，如果因为中途下tensorflow要求的其它安装包而出现错误请自行百度，最后可以用官方提供的一个简单例子看是否成功启动了GPU计算：

```python
import tensorflow as tf

# 新建一个 graph.
a = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[2, 3], name='a')
b = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[3, 2], name='b')
c = tf.matmul(a, b)
# 新建session with log_device_placement并设置为True.
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
# 运行这个 op.
print(sess.run(c))
```

![1531237323389](http://p4mu7u37x.bkt.clouddn.com\GPUsuccess.png)

### 神经网络模型简介

更具体的神经网络模型介绍请参看[这里](http://neuralnetworksanddeeplearning.com/)，这里只针对本题所用到的模型简单谈一下，更主要的目的在于熟悉tensorflow的一些接口。

首先再回顾一下我们将要用到的图片：

<img src="http://p4mu7u37x.bkt.clouddn.com/blog/image/tutorial-kaggle-tensorflow/output_8_1.png" style="margin:0 auto">

图片分宽和高两个方向，两方向尺寸均为96像素，在我们的第一个简单模型中，图片将首先展为96/*96的一维向量，意味着其空间信息将被破坏。这显然不算一个太好的方法，但我们现阶段的目的只是先做个热身而已。

模型分为输入层，隐藏层及输出层，其结构类似于下图：

<img src="https://www.ibm.com/developerworks/cn/java/cc-artificial-neural-networks-neuroph-machine-learning/figure-1.png" style="margin: 0 auto">

当然这里不可能把所有节点都画出，第一层输入层总共有96/*96=9216个节点，第二层隐藏层的数量任意，按照原教程这里取为100，输出层节点数取为30（因为我们总共要输出30个特征）。

### 由tensorflow构建该模型

tensorflow构建该模型的过程可以分为以下几步：

1. 创造占位符；
2. 构建一个完整的前馈神经网络；
3. 添加损失函数；
4. 构建图表并将所有变量初始化；

这里依次说明对应的语法。

```python
"""创造占位符"""

# 图表占位符，作为输入层，None在这里表示任意数量图片，9216即为输入层的节点数，ph代表placeholder
images_ph = tf.placeholder("float", [None, 9216])

# 标记占位符，作为输出层，None与上同理，30表示输出层的节点数
real_labels_ph = tf.placeholder("float", [None, 30])
```

```python
"""构建完整的前馈神经网络"""

# 建立隐藏层作用域，注意Variable只是指定了变量的初始化方法，并未进行初始化
with tf.name_scope("hidden") as scope:
    # 指定输入层到隐藏层权重变量初始化方法（平均值为0，标准偏差为0.01的正态分布，要生成一个9216/*100的权重矩阵），变量名会自动加上作用域名称作为前缀，即"hidden/weights"
    W = tf.Variable(tf.truncated_normal([9216, 100], stddev=0.01), name='weights')
    
    # 指定输入层到隐藏层偏移变量初始化方法（一个大小为100的一维向量，向量所有值均为0）
    b = tf.Variable(tf.zeros([100]), name="biases")
    
    # 输入层与刚刚设定的权重相乘并加上偏差项，激活函数选择rectifier
    hidden = tf.nn.relu(tf.matmul(images_ph, W) + b)
    
# 建立输出层作用域
with tf.name_scope("output") as scope:
    W = tf.Variable(tf.truncated_normal([100, 30], stddev=0.1), name='weights')
    b = tf.Variable(tf.zeros([30]), name="biases")
    
    # 输出层不作处理，即线性函数
    labels_ph = tf.matmul(hidden, W) + b
```

```python
"""添加损失函数"""

# tf.square表示平方，tf.reduce_mean表示求平均，假设对N张训练集中的图片，我们最终通过模型得到了N/*30个预测坐标，则对每个预测坐标，我们可以得到其与真实坐标值的平方差。对所有坐标平方差取平均即为我们的损失函数，模型的目标就是要让这个损失函数最小
loss = tf.reduce_mean(tf.square(labels_ph - real_labels_ph))

# 向模型说明我们要让上述的loss最小，优化方法选AdamOptimizer，学习速率为0.001
train_step = tf.train.AdamOptimizer(1e-3).minimize(loss)
```

```python
"""构建图表并将所有变量初始化"""

# 这条语句可以理解为一个启动按钮，在按下这个按钮后就会将刚刚创建的所有变量(Variable函数)根据其设定的初始化方法进行初始化
init = tf.global_variables_initializer()

# 创建图表
sess = tf.Session()

# run(init)表示按下刚刚设置的启动按钮
sess.run(init)
```

到这步之前，我们完成了所有的准备工作，接着就可以开始训练模型了，这里再多说一句，代码中出现的所有非placeholder及非Variable方法均可以看成一个启动按钮，我们只是设定了这么一个启动按钮，真正要让按钮后面的逻辑运行则需要run方法，正如run(init)所表现的那样。

### 训练模型

在开始训练之前，有一件事得说明一下，让我们再回头看一下整个数据的信息。

```python
# train.info()  没有missingno包的情况下可以直接通过info查看信息，我在这里只是单纯为了可视化

import missingno as msno

msno.matrix(df=train.iloc[:, :-1], figsize=(20, 14))
```

<img src="http://p4mu7u37x.bkt.clouddn.com/missingvalues.png" styles="margin:0 auto">

这张图给出了原始数据的缺省分布，我们从中可以得到一些很有用的信息，这对之后的模型构建至关重要。但在我们的第一个简单模型中，考虑得这么深远有点得不偿失（再次强调，这个模型只是为了熟悉tensorflow的接口）。因此，这一部分的训练数据我们只考虑拥有所有30个完整特征点信息的图，在接下来的分析中你会看到这种图只占了所有训练图的30%，但是没关系，这只是一个起步而已。

```python
full_index = train.dropna().index  # 丢掉null后保留的图片
full_train_input = train_input[full_index]
full_train_output = train_output[full_index]
full_index # Int64Index([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, ... 2272, 2273, 2274, 2275, 2276, 2277, 2278, 2281, 2282, 2283], dtype='int64', length=2140)
```

模型的准备工作已经全部就绪，接着就要开始训练了。

第一步要做的就是要把训练数据划分为训练集和验证集，验证集在这里的作用是调整之后的超参数避免overfitting，这部分用到的函数其实挺好写的，不过sklearn库已经提供了这个功能，能不重复造轮子就不造。

```python
from sklearn.model_selection import train_test_split

# 训练集与验证集如变量名所示，参数中的test_size表示验证集占总比的20%
X_train, X_valid, Y_train, Y_valid = train_test_split(integ_train_input, integ_train_output, test_size=0.2)
```

第二步要做的是构造一个batch函数以期每次迭代都可以从训练集中取出一小部分来训练。这样做的理由有几个：

1. 以整个训练集来训练模型因为计算量的原因迭代往往很慢；
2. 显卡内存有限；

关于batch函数似乎暂时没见到sklearn有相关功能（基本都是在分类器中提供相关参数，可能是内部方法），所以在这里简单写了下：

```python
def batch(X: [list, tuple], batch_size=64, shuffle=True):
    """生成mini_batch，可用于for循环
    
    param: X 要求写成list或tuple类型以应对任意情况
    param: batch_size 每次迭代生成的训练数据量
    param: shuffle 是否对输入进行重排
    """
    n_samples = X[0].shape[0]
    if batch_size > n_samples:
        raise ValueError("batch size is too large")  # batch_size不能大于训练集总的数据量
    index = np.arange(n_samples)
    if shuffle:
        np.random.shuffle(index)
    begin, end = 0, batch_size
    while begin < n_samples:
        yield [e[index[begin:end]] for e in X]  # yield用于之后的for循环
        begin = end
        end = min(end + batch_size, n_samples)
```

第三步就是开始训练啦~

```python
train_loss = list()
valid_loss = list()

# 下面两行只是用来调整输出格式的
print('{:>6}  |{:>12}  |{:>12}  |{:>13}  '.format('Epoch', 'Train loss', 'Valid loss', 'Train / val'))
print('{:->6}--|{:->12}--|{:->12}--|{:->13}--'.format('', '', '', ''))

# 开始迭代
for epoch in range(100):
    for batch_x, batch_y in batch((X_train, y_train)):
        sess.run(train_step, feed_dict={images_ph: batch_x, real_labels_ph: batch_y})
        
    # 这里因为显卡内存不够所以对训练集作了分批处理，也正因为之前编写了batch函数所以很容易就实现啦
    temp_loss = np.zeros(X_train.shape[0])
    for i, (batch_x, batch_y) in enumerate(batch((X_train, y_train), shuffle=False)):
        # 64是我们batch_size的大小
        start, end = i * 64, min((i + 1) * 64, X_train.shape[0])
        temp_loss[start:end] = sess.run(tf.reduce_mean(tf.square(labels_ph - real_labels_ph), 1),
                                        feed_dict={images_ph: batch_x, real_labels_ph: batch_y})
    train_loss.append(np.mean(temp_loss))
    valid_loss.append(sess.run(loss, feed_dict={images_ph: X_valid, real_labels_ph: y_valid}))
    if epoch < 10 or epoch % 10 == 0:
        print('{:>6}  |{:>12.6f}  |{:>12.6f}  |{:>13.6f}  '.format(epoch, train_loss[-1], valid_loss[-1], train_loss[-1] / valid_loss[-1]))
```

得到的损失值如下：

```
 Epoch  |  Train loss  |  Valid loss  |  Train / val  
--------|--------------|--------------|---------------
     0  |    0.105811  |    0.105685  |     1.001199  
     1  |    0.049322  |    0.049157  |     1.003352  
     2  |    0.021933  |    0.021847  |     1.003948  
     3  |    0.012525  |    0.012559  |     0.997250  
     4  |    0.009126  |    0.009267  |     0.984821  
     5  |    0.008095  |    0.008368  |     0.967402  
     6  |    0.010134  |    0.010457  |     0.969111  
     7  |    0.007597  |    0.008056  |     0.942942  
     8  |    0.007370  |    0.007854  |     0.938392  
     9  |    0.007157  |    0.007649  |     0.935620  
    10  |    0.007055  |    0.007572  |     0.931668  
    20  |    0.006222  |    0.006894  |     0.902427  
    30  |    0.007254  |    0.008017  |     0.904818  
    40  |    0.007782  |    0.008596  |     0.905303  
    50  |    0.004654  |    0.005415  |     0.859436  
    60  |    0.004775  |    0.005548  |     0.860695  
    70  |    0.004535  |    0.005255  |     0.863022  
    80  |    0.004342  |    0.005076  |     0.855471  
    90  |    0.003803  |    0.004494  |     0.846304  
```

总得来说似乎还是挺有效的，不妨画张损失的对比图来看一下趋势。

```python
# 这里也可以把画图的几个步骤写成函数方便以后调用
plt.plot(train_loss, label='train')
plt.plot(valid_loss, label='valid')
plt.grid()
plt.legend()
plt.xlabel('epoch')
plt.ylabel('loss')
plt.ylim(min(train_loss) / 1.1, min(max(train_loss), 1e-2))
plt.yscale('log')
plt.show()
```

<img src="http://p4mu7u37x.bkt.clouddn.com/loss.png" style="margin:0 auto">

大致来说可以得出几点结论：

1. 显然100步epoch还没有使模型达到最佳状态
2. 损失下降曲线并不平滑，可能是learning rate取得有点大
3. 训练集与验证集之间的差距在逐渐加大，但两者的趋势还是类似的，还没有严重过拟合

最后再来看一下在测试集上预测的特征点坐标位置：

```python
plt.figure(figsize=(10, 10))
for i in range(9):
    index = int('33{}'.format(i + 1))
    plt.subplot(index)
    plot_sample(test_input[i], test_output[i])
```

<img src="http://p4mu7u37x.bkt.clouddn.com/test.png" style="margin:0 auto">

emmm，效果大概也就那样吧。其实大概也能看到一点端倪，对于非常正的脸，测试效果还是不错的，但对于第一张戴眼镜和第七张脸稍微转了个角度，测试效果一下子就变得马马虎虎了。不过无所谓啦，还是那句话，这个模型只是为了让我们熟悉下接口而已~后面会有更好的模型等着我们。

这篇博客就到这里结束啦，下一期就是正式的神经网络模型了。