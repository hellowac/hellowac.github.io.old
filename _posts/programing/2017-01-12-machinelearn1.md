---
layout: post
title: "机器学习之一 -- k-近邻算法"
author: "Pual"
tags:
  - 'Machine Learn'
  - Learn
category:
  - blog
show: true
---

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
6. **使用算法:** 首先需要输入样本数据和结构化的输出结果，然后运行k-近邻算法判定输入数据分别属于哪个分类，最后应用对计算出的分类执行后续的处理。

## 欧尔距离公式
**计算两个向量点xA和xB之间的距离:**
![距离公式]({% link /assets/programingimg/点距离公式.png %})

如果数据集存在4个特征值，则点(1, 0, 0, 1)与(7, 6, 9, 4)之间的距离计算为:
![距离公式]({% link /assets/programingimg/4点距离公式.png %})


## 分类器

```python
def classify0(inX,dataSet,labels,k,detail=False):
    dataSetSize = dataSet.shape[0]
    
    #距离计算,
    #重复inX数组 dataSetSize次构建一个和dataSet一样大小的数组 c，
    #计算c数组中每个元素和dataSet每个元素的距离
    diffMat = np.tile(inX,(dataSetSize,1)) - dataSet

    sqDiffMat = diffMat**2          #平方去负数
    sqDistances = sqDiffMat.sum(axis=1)     #按1轴统计 -> x 轴
    distances = sqDistances**0.5    #开方, #inX距离每个样本的距离      
    sortedDistIndicies = distances.argsort()  #数组值从小到大的索引排序

    classCount = {}
    
    #选择距离最小的K个点
    for i in range(k):
        #在给定Lebels中投票(距离最近的投票K次)
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel,0) + 1

    #排序
    sortedClassCount = sorted(classCount.iteritems(),
        key=operator.itemgetter(1),reverse=True)

    return sortedClassCount[0][0]

#使用Matplotlib创建散点图
#归一化特征值:
#公式: newValue = (oldValue-min)/(max-min)
def autoNorm(dataSet,detail=False):
    '''
        dataSet 为 numpy.ndarray类型
    '''
    maxVals = dataSet.max(0)        #坐标系第0轴-Y,1x3的矩阵值
    minVals = dataSet.min(0)        #坐标系第0轴-Y,1x3的矩阵值

    #(max-min)
    ranges = maxVals - minVals

    normDataSet = np.zeros(np.shape(dataSet))
    m = dataSet.shape[0]

    normDataSet = dataSet - np.tile(minVals,(m,1))      #1x3 -> 1000x3 的矩阵值

    #(oldValue-min)/(max-min) , 矩阵除法->linalg.solve(matA,matB) [mat -> matrix]
    normDataSet = normDataSet/np.tile(ranges,(m,1))     #1x3 -> 1000x3 的矩阵值

    return normDataSet,ranges,minVals

#创建扩散图(散点图):
def plot_test(datingDataMat,datingLabels):
    #创建一幅图
    fig = plt.figure()

    # add_subplot(3,3,5)  等价于 add_subplot(335) ,表示将画布分割成3行3列，图像画在从左到右从上到下的第5块
    #添加小图
    ax = fig.add_subplot(111)  #等价于add_subplot(1,1,1)

    #http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.scatter
    # ax.scatter(x,y,**kwargs)

    #散点/扩散图, 加颜色标示(粒度比较小)
    ax.scatter(datingDataMat[:,0],
        datingDataMat[:,1],
        c=15.0*np.array(datingLabels),      #颜色
        s=15.0*np.array(datingLabels),      #大小
        alpha=0.7                                       #透明度
        )

    plt.show()
```

## 总结
* **优点:** 精度高、对异常值不敏感、无数据输入假定。
* **缺点:** 计算复杂度高、空间复杂度高。
* **适用数据范围:** 数值型和标称型。