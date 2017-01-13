---
nav: blog
layout: post
title: "机器学习之一 -- k-近邻算法"
author: "Pual"
tags:
  - 'Machine Learn'
  - Learn
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

# K-近邻算法
----------------
**k-近邻算法采用测量不同特征值之间的距离方法进行分类。**

## 工作原理:
* 存在一个样本数据集合,也称作`训练样本集`,并且 **样本集** 中每个数据都存在`标签`,即我们知道 **样本集** 中每一数据与所属分类的 **应对关系** 。
* 输入没有标签的 **新数据** 后，将 **新数据** 的每个`特征`与样本集中数据对应的`特征`进行比较，然后算法提取 **样本集** 中特征 最相似 数据(**最近邻**)的`分类标签`。
* 一般来说，只选择 **样本数据** 集中前`K`个最相似的数据，这就是`k-近邻算法`中k的出处，通常`K`是 **不大于20** 的整数。最后选择`K`个最 **相似数据** 中 **出现次数** 最多的分类，作为新数据的`分类`。

## 一般流程:
1. **收集数据:** 可以使用任何方法.
2. **准备数据:** 距离计算所需要的数值，最好是结构化的数据格式。
3. **分析数据:** 可以使用任何方法。
4. **训练算法:** 此步骤不适用于k-近邻算法。
5. **测试算法:** 计算错误率。
6. **使用算法:** 首先需要输入 __样本数据__ 和结构化的输出结果，然后运行`k-近邻算法`判定输入数据分别属于哪个`分类`，最后应用对计算出的分类执行后续的处理。

## 欧尔距离公式
**计算两个向量点xA和xB之间的距离:**
![距离公式]({% link /assets/programingimg/点距离公式.png %})

如果数据集存在4个特征值，则点(1, 0, 0, 1)与(7, 6, 9, 4)之间的距离计算为:
![距离公式]({% link /assets/programingimg/4点距离公式.png %})


## 创建数据
```python
import numpy as np

def createDataSet():
    group = np.array([
                    [1.0,1.1],
                    [1.0,1.0],
                    [0,0],
                    [0,0.1],
        ])
    labels = ['A','A','B','B']
    
    return group,labels
```

## 分类器函数
__将一组数据按照样本集分类__
```python
    
def classify0(inX,dataSet,labels,k):
    """分类器.

    将指定的数据集合进行分类

    参数:
        inX:要分类的一维数组（长度必须和样本集中列数相同）
        dataSet:numpy.ndarry类型的集合 样本集
        labels:样本集每组数据对应的标签(一位数组)
        k:指定采用多少前样本集依据

    返回值:
        该数据集的标签.

    异常:
        皆有可能。
    """

    # 找出 样本集 的行数.
    # shape参考:https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.shape.html
    dataSetSize = dataSet.shape[0]
    
    # 距离计算,
    # 重复inX数组 dataSetSize次 构建一个和dataSet一样大小的数组 c，
    # 计算c数组中每个元素和dataSet每个元素的距离
    # tile参考:https://docs.scipy.org/doc/numpy/reference/generated/numpy.tile.html
    diffMat = np.tile(inX,(dataSetSize,1)) - dataSet

    # 平方去负数
    sqDiffMat = diffMat**2

    # 按1轴统计 -> x 轴
    sqDistances = sqDiffMat.sum(axis=1)

    # 开方, inX 距离每个 样本 的距离
    distances = sqDistances**0.5    

    # 数组值从小到大的索引
    # argsort 参考:https://docs.scipy.org/doc/numpy/reference/generated/numpy.argsort.html
    sortedDistIndicies = distances.argsort()  

    from collections import defaultdict

    classCount = defaultdict(lambda : 0)
    
    # 选择距离最小的K个点
    for i in range(k):

        # 在给定Lebels中投票(距离最近的投票K次)
        voteIlabel = labels[sortedDistIndicies[i]]

        classCount[voteIlabel] += 1

    import operator

    #排序
    sortedClassCount = sorted(classCount.iteritems(),
        key=operator.itemgetter(1),reverse=True)

    return sortedClassCount[0][0]
```

### 练习1
```python
#常规数据创建
group,labels = createDataSet()

#分类事例
category = classify0([1,0],group,labels,3)

print category
```
__结果:__
```
B
[Finished in 0.4s]
```

## 归一化数值
__将大值取值范围处理为0到1或者-1到1之间。__

__公式:__ `newValue = (oldValue-min)/(max-min)`

**现有一组这样的数据:**

----|-----|----|------
40920 | 8.326976 | 0.953952 | 3
14488 | 7.153469  | 1.673904 | 2
26052 | 1.441871  | 0.805124 | 1
75136 | 13.147394 | 0.428964 | 1
38344 | 1.669788  | 0.134296 | 1
72993 | 10.141740 | 1.032955 | 1
35948 | 6.830792  | 1.213192 | 3
42666 | 13.276369 | 0.543880 | 3
67497 | 8.631577  | 0.749278 | 1
35483 | 12.273169 | 1.508053 | 3


**归一化数值后为:**

----|-----|----|------
0.43582641 | 0.5817826  | 0.53237967 | 3
0.         | 0.48262275 | 1.         | 2
0.19067405 | 0.         | 0.43571351 | 1
1.         | 0.98910178 | 0.19139157 | 1
0.3933518  | 0.0192587  | 0.         | 1
0.96466495 | 0.73512784 | 0.58369338 | 1
0.35384514 | 0.45535696 | 0.70076019 | 3
0.46461549 | 1.         | 0.26603135 | 3
0.87404366 | 0.60752099 | 0.39944064 | 1
0.34617794 | 0.91523088 | 0.89227713 | 3

__归一化函数__:
```python
def autoNorm(dataSet):
    """归一化数据函数:

    将任意取值范围的特征值转化为0到1区间内的值
    公式: newValue = (oldValue-min)/(max-min)
        
    参数:
        dataSet: numpy.ndarray类型

    返回值:
        dataset:值为0-1和-1至1之间的值.
        ranges:最大值到最小值之间的差
        minvals:最小值
    """

    #坐标系第0轴-Y,1x3的矩阵值
    maxVals = dataSet.max(0)
    
    #坐标系第0轴-Y,1x3的矩阵值
    minVals = dataSet.min(0)        

    #(max-min)
    ranges = maxVals - minVals

    #一个tuple类型.长度为1时表示1纬，并且表示元素个数，
    #长度为2时表示2纬，[0] 行,[1]列数据
    #当每行数据不相等时，当1纬处理返回[0]行数据
    m = dataSet.shape[0]

    #重复minVals数组m次,累加1次，
    #--> array([minVals,minVals,...,minVals]) 
    #--> array([[minVal...],[minVal...],...[minVal]])
    #(oldValue-min)
    normDataSet = dataSet - np.tile(minVals,(m,1))      #1x3 -> 1000x3 的矩阵值

    #(oldValue-min)/(max-min) , 矩阵除法->linalg.solve(matA,matB) [mat -> matrix]
    normDataSet = normDataSet/np.tile(ranges,(m,1))     #1x3 -> 1000x3 的矩阵值

    return normDataSet,ranges,minVals
```

### 练习2:
```python
array = [
    [40920,8.326976 , 0.953952, ],
    [14488,7.153469 , 1.673904, ],
    [26052,1.441871 , 0.805124, ],
    [75136,13.147394, 0.428964, ],
    [38344,1.669788 , 0.134296, ],
    [72993,10.141740, 1.032955, ],
    [35948,6.830792 , 1.213192, ],
    [42666,13.276369, 0.543880, ],
    [67497,8.631577 , 0.749278, ],
    [35483,12.273169, 1.508053, ],
]

narray = np.array(array)
normMat,ranges,minVals = kNN.autoNorm(narray)

print normMat
```

__结果:__
```
[[ 0.43582641  0.5817826   0.53237967]
 [ 0.          0.48262275  1.        ]
 [ 0.19067405  0.          0.43571351]
 [ 1.          0.98910178  0.19139157]
 [ 0.3933518   0.0192587   0.        ]
 [ 0.96466495  0.73512784  0.58369338]
 [ 0.35384514  0.45535696  0.70076019]
 [ 0.46461549  1.          0.26603135]
 [ 0.87404366  0.60752099  0.39944064]
 [ 0.34617794  0.91523088  0.89227713]]
[Finished in 0.4s]
```

## 使用matplotlib绘图（点散图）
参考:<http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.scatter>

```python
import matplotlib.pyplot as plt

def plot_test(datingDataMat,datingLabels):
    #创建一幅图
    fig = plt.figure()

    # add_subplot(3,3,5)  等价于 add_subplot(335) ,表示将画布分割成3行3列，图像画在从左到右从上到下的第5块
    #添加小图
    ax = fig.add_subplot(111)  #等价于add_subplot(1,1,1)

    # print 15.0*np.array(datingLabels)

    # ax.scatter(datingDataMat[:,1],datingDataMat[:,2])

    #http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.scatter
    # ax.scatter(x,y,**kwargs)

    #散点/扩散图, 加颜色标示(粒度比较小)
    ax.scatter(datingDataMat[:,0],
        datingDataMat[:,1],
        c=15.0*np.array(datingLabels),      #颜色
        s=15.0*np.array(datingLabels),      #大小
        alpha=0.7                           #透明度
        )

    #显示
    plt.show()
```

### 练习3 分类并绘图
```python
def f1():

    # 样本集
    array = [
        [40920,8.326976 , 0.953952, ],
        [14488,7.153469 , 1.673904, ],
        [26052,1.441871 , 0.805124, ],
        [75136,13.147394, 0.428964, ],
        [38344,1.669788 , 0.134296, ],
        [72993,10.141740, 1.032955, ],
        [35948,6.830792 , 1.213192, ],
        [42666,13.276369, 0.543880, ],
        [67497,8.631577 , 0.749278, ],
        [35483,12.273169, 1.508053, ],
    ]

    # 样本集标签
    arrayLabels = [3, 2, 1, 1, 1, 1, 3, 3, 1, 3]

    narray = np.array(array)
    narrayLabels = np.array(arrayLabels)

    # 分类数据集
    inxs = [
        [ 69673,   14.239195,   0.261333],
        [ 15669,   0.000000 ,   1.250185],
        [ 28488,   10.528555,   1.304844],
        [ 6487 ,   3.540265 ,   0.822483],
        [ 37708,   2.991551 ,   0.833920],
        [ 22620,   5.297865 ,   0.638306],
        [ 28782,   6.593803 ,   0.187108],
        [ 19739,   2.816760 ,   1.686209],
    ]

    inxs = np.array(inxs)

    # 归一化数据:
    normMat,ranges,minVals = autoNorm(inxs)

    classfy = []

    # 分类该数据集
    for inx in inxs :
        label = classify0(inx, narray, narrayLabels, 5)
        classfy.append(label)

    # 绘制分类的数据集:
    plot_test(normMat,classfy)

if __name__ == '__main__':
    f1()
```

__结果如图:__

![matplotlib绘图]({% link assets/programingimg/k-近邻算法.png %})

## k-近邻算法 总结
* **优点:** 精度高、对异常值不敏感、无数据输入假定。
* **缺点:** 计算复杂度高、空间复杂度高。
* **适用数据范围:** 数值型和标称型。