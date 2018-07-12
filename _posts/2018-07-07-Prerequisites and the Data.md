---
layout: post
title: Prerequisites and the Data
category: Machine Learning
permalink: /blog/Machine Learning/:title
---

作为机器学习博客的第一篇，我想将其写为kaggle和tensorflow的一个入门实践，其中会涉及到jupyter的基本运用，常用python包的基本介绍，深度神经网络构建及tensorflow的初级运用。这里所采用的案例是kaggle的入门级比赛[facial keypoints detection](https://www.kaggle.com/c/facial-keypoints-detection)，官方已经提供了一个基于Lasagne及Theano的[教程](http://danielnouri.org/notes/2014/12/17/using-convolutional-neural-nets-to-detect-facial-keypoints-tutorial/)，而这个系列博客的绝大部分工作基本都是在将原博的代码框架转到tensorflow。我会按照下面所列的目录顺序来写。

* [赛题介绍及一些预备知识]()
* [一个简单的神经网络模型]({{baseurl}}/blog/{{page.category}}/First-Model-A-Single-Hidden-Layer)
* [卷积神经网络模型构建]()
* [卷积神经网络的完善与强化]()
* [结论]()

## 赛题介绍及一些预备知识

### 关于kaggle

kaggle成立于2010年，是一个进行数据挖掘和预测竞赛的在线平台。从公司角度讲，它可以在平台上提出一个实际需要解决的问题并提供相关数据及奖金；从参赛者角度讲，可以针对问题组队参与项目并提出解决方案以期赢取奖金。

kaggle的竞赛有多种分类，包括featured，research，getting started等。各种分类的比赛模式都是一样的，即通过出题方给予的训练集建立模型，再利用测试集算出结果上传平台用以评分。在比赛截止日期前，所有队伍都可加入比赛，并提交或改善自己的模型结果，同时你可以在比赛的排行榜上看到自己的排名。

kaggle的一大特色就是kernel，与jupyter的功能类似（jupyter会在之后稍微介绍一下）。在这里有一些参赛者会写下他们的建模思路以及建模过程，其中不少还包含源码。得益于平台的良好支持，点赞数高的kernel往往拥有良好的文档结构及精美的图表，新手往往能从这些kernel中获得不少收获。

由于平台在计算排名上只需要你提供最终的预测结果，所以你用什么语言来进行建模都无所谓。不过绝大部分kernel均用python或者R编写，所以最好还是至少熟悉其中一种语言。

### 关于jupyter

通常我们会在pycharm中编写自己的项目，不过对于数据分析而言，一个可以实时交互的IDE才是最理想的选择，jupyter采用类似matlab分节运行的方式及魔法命令完美满足了这个功能。jupyter的本质是一个web应用程序（意味着你可以在你的浏览器中使用它），可以通过代码实时生成图片、视频等，同时也支持LaTeX和Markdown。

<img src="http://p4mu7u37x.bkt.clouddn.com/jupyterpreview.png" style="margin:0 auto">

更多jupyter的介绍请见[官网](https://jupyter.org/)

### 关于本次赛题

这篇博文选取的赛题是getting started级别的入门比赛——facial keypoints detection，直译过来就是脸部特征识别。选择这场比赛的主要原因是其有完整的官方教程，而官方教程基于Lasagne及Theano，所以，将教程所用到的方法转移到tensorflow下的框架似乎是一个很好的练习。比赛提供了几千张图片的灰度值及一些特征信息，我们首先要做的就是对这些数据进行一些可视化处理。在这里我会提供代码及部分注释及其展示结果，之后的分析也将同样遵循这个流程。

```python
import pandas as pd
train = pd.read_csv('./data/training.csv')  # 训练集数据所在路径
train.head()
```

pandas可以说是python中用于数据分析的专用库了，具体介绍请见[官网](https://pandas.pydata.org/)。上述代码中train属于pandas中的DateFrame类，调用head方法可以获取train中前几张图片的信息，并可以通过jupyter的支持良好地在网页中展现出来。

|      | left_eye_center_x | left_eye_center_y | right_eye_center_x | right_eye_center_y | ...  | Image                     |
| ---- | ----------------- | ----------------- | ------------------ | ------------------ | ---- | ------------------------- |
| 0    | 66.033564         | 39.002274         | 30.227008          | 36.421678          | ...  | 238 236 237 238 240 24... |
| ...  |                   |                   |                    |                    |      |                           |

不失普遍性，这里截取了第一张图片的部分数据。从第二列开始到倒数第二列，列名均为图片的特征（像是left_eye_center_x, left_eye_center_y等），其值则为特征点的坐标。最后一列的列名为Image，列出了图像的灰度信息。

由于这些数据本身就表示图片，在这里我们可以更直观一些，直接还原该图片。

```python
import matplotlib.pyplot as plt
import numpy as np

def preprocess(x):
    """将所有灰度值转为0-1的数值"""
    return np.array(list(map(int, x.split()))) / 255

first_image = train[0:1]  # 需注意pandas切片操作
img = preprocess(train['Image'][0])
features = train[train.columns[:-1]].values[0]
img = img.reshape((96, 96))  # 图像的尺寸大小为96*96
plt.imshow(img, cmap='gray')
plt.scatter(features[0::2], features[1::2], marker='x')  # 标出特征点
```

上述代码涉及到numpy与matplotlib库的运用，前者为python科学计算必备，后者为python的画图库，具体不展开。总的来说这几行代码的意思为提取第一张图片的信息，对灰度值进行处理转为一个0-1的数值向量。然后我们将该向量重设为96*96的数值矩阵，就可以通过matplotlib展示该数值矩阵对应的图片了。除此之外我们还将该图的所有特征点位置在图上标了出来，最后的图片如下所示。

<img src="http://p4mu7u37x.bkt.clouddn.com/blog/image/tutorial-kaggle-tensorflow/output_8_1.png" style="margin:0 auto">

通过图片就很直观了，我们的最终目的就是构件一个模型，以训练集中每张图片的灰度数值作为输入，特征点坐标作为输出训练该模型，然后将该模型运用在测试集上验证准确率。

在这里为了之后提取与处理数据方便起见，我们可以将上面设计到的一些操作写为函数存入util.py中，这样，之后再用到这些功能时只要调用就可以了（这么做的最大原因还是让你的精力可以集中在分析数据上）。

```python
def load_train():
    train = pd.read_csv('./data/training.csv')
    x = np.vstack(train['Image'].apply(preprocess))
    y = (train[train.columns[:-1]].values - 48) / 48
    return x, y

def load_test():
    test = pd.read_csv('./data/test.csv')
    return np.vstack(test['Image'].apply(preprocess))

def plot_sample(image, features):
    img = image.reshape((96, 96))
    plt.imshow(img, cmap='gray')
    plt.scatter(features[0::2] * 48 + 48, features[1::2] * 48 + 48, marker='x')
```

需要注意以下几点：

- load_train返回x与y作为训练数据，x作为输入值，y作为输出值，我们需要通过这些来训练模型；
- 对于load_train得到的y值，我们进行了归一化处理（图片原大小为96*96），同样的原因，在绘制时需要将坐标还原为真实坐标
- 在训练完模型后，load_test得到测试集，我们将测试集传入模型并预测特征点坐标；

在这篇博文的最后，让我们来看一下最终的输入输出值顺便测试一下编写的函数。

```python
train_input, train_output = load_train()
test_input = load_test()
plt.subplot(121)  # 表示当前图层为两个子图中的第一个子图
plot_sample(train_input[0], train_output[0])
plt.subplot(122)  # 表示当前图层为两个子图中的第二个子图
plot_sample(train_input[1], train_output[1])
```

<img src="http://p4mu7u37x.bkt.clouddn.com/blog/image/tutorial-kaggle-tensorflow/output_16_0.png" style="margin:0 auto">

参照原教程，这个系列的第二篇文章会通过tensorflow建立一个简单的神经网络模型，正式开始我们的tensorflow之旅啦~