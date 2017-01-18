---
nav: blog
layout: post
title: "机器学习 -- 决策树"
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

# 决策树
----------------

## 一般流程:
1. __收集数据:__ 可以使用任何方法。 
2. __准备数据:__ 树构造算法只适用于标称型数据,因此数值型数据必须离散化。 
3. __分析数据:__ 可以使用任何方法,构造树完成之后，我们应该检查图形是否符合预期。 
4. __训练算法:__ 构造树的数据结构。 
5. __测试算法:__ 使用经验树计算错误率。 
6. __使用算法:__  此步骤可以适用于任何监督学习算法， 使用决策树可以更好地理解数据的内在含义.

## 信息增益:

将无序的数据变得更加有序

__参考:__ [香农.熵](https://www.baidu.com/link?url=TcEPIAxlk8pxNioBJXM0r_yf8dDY9sITv_lDZf4RFyalzHNhzL3dSxusPSSmAZJRSv4svozev4XjOYtakDIpWtNrycnEBaIKeDWsKM93BDvv1YDklCanVdrZwNbBdFDv&wd=&eqid=af6cba170000101500000002587d9ec0)

计算所有类别所有可能包含的信息期望值.

公式:

![熵]({% link assets/programingimg/香农.熵.png %})

其中的p(x)是选择该分类的概率.

## 函数:计算给定数据集的香农熵.(计算无序程度)
```python
import math

def calcShannonEnt(dataSet):
    """计算香农.熵

    返回最佳分隔符,公式:http://blog.sina.com.cn/s/blog_7004135d01011j2f.html

    参数:
        dataset:必须是一种由列表元素组成的列表,
        并且所有的列表元素具有相同的数据长度，
        数据的最后一列或每个实例的最后一个元素是当前实例的类别标签.
        列表中的数据类型,可以是数字也可以是字符串，并不影响实际运算.

    返回值:
        float:香农.熵
    
    异常:
        皆有可能.
    """
    numEntries = len(dataSet)
    labelCounts = {}
    for featVec in dataSet :

        # 取类别标签
        currentLabel = featVec[-1]

        if currentLabel not in labelCounts.keys():
            labelCounts[currentLabel] = 0

        labelCounts[currentLabel] +=1 

    shannonEnt = 0.0
    for key in labelCounts :

        #选择该分类的概率 [p(x)]
        prob = float(labelCounts[key])/numEntries

        shannonEnt -= prob*math.log(prob,2)

    return shannonEnt
```

* __实践:__

__创建数据:__

```python
def createDataSet():
    """最后一列为结论
    """
    #是否属于鱼类
    labels = [u'不换气?',u'有脚蹼?']

    #条件,结论
    dataSet = [
        [u'是',u'是',u'属于'],
        [u'是',u'是',u'属于'],
        [u'是',u'否',u'不属于'],
        [u'否',u'是',u'不属于'],
        [u'否',u'是',u'不属于'],
    ]

    return dataSet,labels
```

__计算熵:__

```python
In [15]: data,labels = createDataSet()

In [77]: data
Out[77]:
[[u'是',u'是',u'属于'],
    [u'是',u'是',u'属于'],
    [u'是',u'否',u'不属于'],
    [u'否',u'是',u'不属于'],
    [u'否',u'是',u'不属于'],]

In [78]: trees.calcShannonEnt(data)
Out[78]: 0.9709505944546686

In [82]: mydata[0][-1] = u'可能'

In [84]: trees.calcShannonEnt(mydata)
Out[84]: 1.3709505944546687
```

- 另一种 `计算无序程度` 方法: __基尼不纯度__(Gini impurity)
- 从一个数据集中随机抽取子项,度量其被错误分类到其他分组里的概率
- 参考:<http://blog.verypod.com/cart%E7%AE%97%E6%B3%95%E4%B8%AD%E7%9A%84gini-impurity%EF%BC%88%E4%B8%8D%E7%BA%AF%E5%BA%A6%EF%BC%89/>

## 划分数据集:判断按照那个特征划分数据集是最好的划分方式.

__划分数据:__

```python
def splitDataSet(dataSet,axis,value):
    """划分数据集.

    参数:
        dataSet:带划分的数据集,需满足计算 熵 的 数据集合.[等列,最后一列为类别标签]
        axis:划分数据集的特征的列索引.
        value:需要返回的特征值.

    返回值:
        列表:

    """
    retDataSet = []
    for featVec in dataSet :
        if featVec[axis] == value :
            reducedFeatVec = featVec[:axis]
            reducedFeatVec.extend(featVec[axis+1:])
            retDataSet.append(reducedFeatVec)

    return retDataSet
```

* __实践__:

__划分数据:__

```python
In [27]: data,labes = trees.createDataSet()

In [85]: splitDataSet(mydata,0,u'否')
Out[85]: [[u'是', u'不属于'], [u'是', u'不属于']]

In [86]: splitDataSet(mydata,0,u'是')
Out[86]:
[[u'是', u'属于'],
 [u'是', u'属于'],
 [u'否', u'不属于']]
```

__选择最好的划分方式:__

```python
def chooseBestFeatureToSplit(dataSet):
    '''选择最好的数据划分方式
    
    参数:
        dataSet:等列的二维数据集
            1.数据必须是一种由列表元素组成的列表,而且所有的列表元素都要具有相同的数据长度
            2.数据的最后一列或者每个实例的最后一个元素是当前实例的类别标签
            满足以上要求后,便可以判断包含多少特征属性，
            无须限定list中的数据类型,既可以是数字也可以是字符串

    返回值:
        最好的特征值下标.
    '''
    # feature:特征
    # gain:收益
    # entropy:熵

    # 最后一个元素为类别标签
    numFeatures = len(dataSet[0]) - 1 

    # 原始香农.熵,用于比较
    baseEntropy = calcShannonEnt(dataSet)

    bestInfoGain = 0.0
    bestFeature = -1

    for i in range(numFeatures) :

        # 创建特征值列表,按列取值.
        featList = [example[i] for example in dataSet ] 

        uniqueVals = set(featList)
        newEntropy = 0.0

        for value in uniqueVals :
            # 对每个特征划分一次数据集
            subDataSet = splitDataSet(dataSet,i,value)

            # 概率
            prob = len(subDataSet)/float(len(dataSet))

            # 新香农.熵,
            newEntropy += prob*calcShannonEnt(subDataSet)

        # 信息增益,熵的减少或者是数据无序度的减少(熵用于度量数据的无序度)
        infoGain = baseEntropy - newEntropy

        if (infoGain > bestInfoGain) :
            bestInfoGain = infoGain
            bestFeature = i

    return bestFeature
```

* __实践__:

__最好的特征值:__

```python
In [87]: mydata,labels = createDataSet()

In [88]: chooseBestFeatureToSplit(mydata)
Out[88]: 0      #选择下标为0的为最好的特征

In [89]: mydata
Out[89]:
[[u'是',u'是',u'属于'],
    [u'是',u'是',u'属于'],
    [u'是',u'否',u'不属于'],
    [u'否',u'是',u'不属于'],
    [u'否',u'是',u'不属于'],]
```

## 递归构建决策树
__选择最好的特征:__

```python
import operator
from collections import defaultdict
def majorityCnt(classList):
    """选择最好的分类.
    
    参数:
        classList:分类特征值列表
    
    返回值:
        最好的特征值
    """
    classCount = defaultdict(lambda : 0)

    for vote in classList :
        classCount[vote] += 1

    sortedClassCount = sorted(classCount,key=operator.itemgetter(1),reverse=True)

    return sortedClassCount[0][0]
```

__构建树:__

```python
import copy
def createTree(dataSet,labels):
    """创建树

    创建字典类型的决策树

    参数:
        dataSet:数据集,[等列,最后一位为特征值]
        labels:特征值列表.

    返回值:
        dict:决策树
    """
    labels = copy.copy(labels)

    # 类别标签
    classList = [example[-1] for example in dataSet]

    if classList.count(classList[0]) == len(classList):
        return classList[0]

    if len(dataSet[0]) == 1 :
        return majorityCnt(classList)

    # 最好的特征下标
    bestFeat = chooseBestFeatureToSplit(dataSet)
    bestFeatLabel = labels[bestFeat]

    myTree = {bestFeatLabel:{}}

    del(labels[bestFeat])

    featValues = [example[bestFeat] for example in dataSet]

    uniqueVals = set(featValues)

    for value in uniqueVals:
        subLabels = labels[:]

        subDataSet = splitDataSet(dataSet, bestFeat, value)

        myTree[bestFeatLabel][value] = createTree(subDataSet, subLabels)

    return myTree
```

* __实践:__

__创建树:__

```python

In [36]: mydata,labels = createDataSet()

In [91]: myTree = createTree(mydata,labels)

In [92]: myTree
# 转换后:
Out[92]:
{
    "不换气?":{
        "是":{
            "有脚蹼?":{
                "是":"属于",
                "否":"不属于"
            }
        },
        "否":"不属于"
    }
}
```

## 绘制树形图[__有难度__]

__绘制节点:__

```python
import matplotlib.pyplot as plt
decisionNode = dict(boxstyle="sawtooth", fc="0.8")
leafNode = dict(boxstyle="round4", fc="0.8")
arrow_args = dict(arrowstyle="<-")
def plotNode(plot,nodeTxt, centerPt, parentPt, nodeType):
    """绘制节点

    参数:
        plot:图.
        nodeTxt:注释文字.
        centerPt:起始坐标.
        parentPt:结束坐标.
        nodeType:节点类型.

    返回值:
        None

    """
    # 参考: http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.annotate
    # 在图上添加注释
    plot.axl.annotate(nodeTxt,
                            xy=parentPt, 
                            xycoords='axes fraction',
                            xytext=centerPt, 
                            textcoords='axes fraction',
                            va="center", 
                            ha="center", 
                            bbox=nodeType,
                            arrowprops=arrow_args)
```
__测试绘制图:__

```python
def createPlot_test():
    """测试绘制节点函数.
    """

    # 参考:http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.figure
    fig = plt.figure(1, facecolor='white')

    # 参考:http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.clf
    fig.clf()
    # 参考:http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.subplot
    createPlot_test.axl = plt.subplot(111, frameon=False)
    plotNode(createPlot_test,u'决策树', (0.5, 0.1), (0.1, 0.5), decisionNode)
    plotNode(createPlot_test,u'叶子', (0.8, 0.1), (0.3, 0.8), decisionNode)
    plt.show()
```
![测试节点绘图]({% link assets/programingimg/决策树绘制节点测试.png %})
不能显示中文？参考:<http://www.oucb.org/archives/511/>和<http://www.pythoner.com/200.html>

__获取叶子节点数:__

```python
def getNumLeafs(myTree):
    """获取叶节点数

    也就是树的宽度

    参数:
        myTree:json格式的树结构

    返回值:
        int:叶节点数.
    """
    numLeafs = 0
    firstStr = myTree.keys()[0]
    secondDict = myTree[firstStr]

    for k,v in secondDict.iteritems():
        if type(v) is dict :
            numLeafs += getNumLeafs(v)
        else :
            numLeafs += 1

    return numLeafs
```

__获取树的深度:__

```python
def getTreeDepth(myTree):
    """获取树的层数

    也就是树的深度

    参数:
        myTree:json格式的树结构。

    返回值:
        int:树的深度.
    """
    maxDepth = 0
    firstStr = myTree.keys()[0]
    secondDict = myTree[firstStr]

    for k,v in secondDict.iteritems():

        if type(v) is dict:
            thisDepth = 1 + getTreeDepth(v)
        else:
            thisDepth = 1

        if thisDepth > maxDepth:
            maxDepth = thisDepth

    return maxDepth
```
__创建测试数据树:__

```python
def retrieveTree(i):
    """根据下标获取数据.
    """
    listOfTrees = [
        {
            u"不换气?":{
                u"是":{
                    u"有脚蹼?":{
                        u"是":u"属于",
                        u"否":u"不属于"
                    }
                },
                u"否":u"不属于"
            }
        },{
            u"不换气?":{
                u"是":{
                    u"有脚蹼?":{
                        u"是":u"不属于",
                        u"否":{
                            u"有头?":{
                                u"否":u"不属于",
                                u"是":u"属于"
                            }
                        }
                    }
                },
                u"否":u"不属于"
            }
        }
    ]
    return listOfTrees[i]
```
__填充中间文本:__

```python
def plotMidText(plot,strPt,parentPt,txtString):
    """填充中间的文本.
    
    在父子节点间填充文本信息

    参数：
        plot:图.
        strPt:起始坐标.
        parentPt:结束坐标.
        txtString:填充的文本.

    返回值:
        无
    """
    xMid = (parentPt[0] - strPt[0]) / 2.0 + strPt[0]
    yMid = (parentPt[1] - strPt[1]) / 2.0 + strPt[1]

    # 向图中添加文本信息(坐标,信息)
    plot.axl.text(xMid,yMid,txtString)
```
__绘制树:__

```python
def plotTree(myTree,parentPt,nodeTxt):
    """绘制树形图

    绘制树形图:
    1.计算树的宽和高.
    2.totalW存储树的宽度,totalD存储树的深度,这两个计算树节点的摆放位置.
    3.xOff和yOff追踪已经绘制的节点位置.以及下一个节点的恰当位置.
    4.通过计算树包含的所有叶子节点数,划分图形的宽度.从而计算得到当前节点的中心位置.

    参数:
        myTree:json格式的树结构
        parentPt:结束坐标.
        nodeTxt:注释文字.

    返回值:
        无
    """
    # 叶子数-宽度
    numLeafs = getNumLeafs(myTree)
    # 深度 - 高度
    depth = getTreeDepth(myTree)

    firstStr = myTree.keys()[0]
    strPt = (plotTree.xOff + (1.0 + float(numLeafs))/2.0/plotTree.totalW,plotTree.yOff)

    # 注释父子节点间文本 [主分支的注释]
    plotMidText(createPlot, strPt, parentPt, nodeTxt)

    # 绘制节点 和 树干 [主分支和父节点]
    plotNode(createPlot, firstStr, strPt, parentPt, decisionNode)

    secondDict = myTree[firstStr]

    plotTree.yOff = plotTree.yOff - 1.0 / plotTree.totalD

    for k,v in secondDict.iteritems():

        # 为dic时继续绘制
        if type(v) is dict:
            plotTree(v, strPt, k)

        else :
            plotTree.xOff = plotTree.xOff + 1.0 / plotTree.totalW

            # 绘制节点 和 树干 [次分支和子节点]
            plotNode(createPlot,v, 
                    (plotTree.xOff,plotTree.yOff), 
                    strPt, leafNode)

            # 注释父子节点间文本 [次分支的注释]
            plotMidText(createPlot,(plotTree.xOff,plotTree.yOff), strPt , k)

    plotTree.yOff = plotTree.yOff + 1.0 / plotTree.totalD
```
__创建图:__

```python
def createPlot(inTree):
    
    # 参考:http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.figure
    #plt.figure:绘图组件的顶层容器。
    fig = plt.figure(1,facecolor='white')

    # 参考:http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.clf
    fig.clf()

    axprops = dict(xticks=[],yticks=[])
    
    # 参考:http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.subplot
    # plt.subplot : 创建一个字图,行数,列数,序号,
    createPlot.axl = plt.subplot(111,frameon=False,**axprops)

    plotTree.totalW = float(getNumLeafs(inTree))
    plotTree.totalD = float(getTreeDepth(inTree))
    plotTree.xOff = -0.5/plotTree.totalW
    plotTree.yOff = 1.0

    parentPt = (0.5,1.0)
    nodeTxt = ''
    plotTree(inTree, parentPt, nodeTxt)

    plt.show()
```
* __实践:__

* __绘制树:__

```python
In [37]: mytree = retrieveTree(1)

In [38]: createPlot(mytree)
```
是否属于鱼类:
![决策树图]({% link assets/programingimg/决策树图.png %})
![决策树图]({% link assets/programingimg/决策树图2.png %})

## 测试和存储分类器
__分类器:__

```python
def classify(inTree,featLabels,testVec):
    """决策树分类

    参数:
        inTree:树
        featLabels:特征类型列表
        testVec:测试的数据列表.

    返回值:
        int or str :分类结果
    """
    firstStr = inTree.keys()[0]

    secondDict = inTree[firstStr]

    featIndex = featLabels.index(firstStr)

    key = testVec[featIndex]

    valueOfFeat = secondDict[key]

    if isinstance(valueOfFeat, dict): 
        classLabel = classify(valueOfFeat, featLabels, testVec)
    else: 
        classLabel = valueOfFeat

    return classLabel
```
* __实践:__

__测试分类器:__

```python
In [72]: mydata,lebels = createDataSet()

In [73]: myTree = createTree(mydata,labels)

In [74]: labels
Out[74]: [u'\u4e0d\u6362\u6c14?', u'\u6709\u811a\u8e7c?']
#转换后:
Out[74]: ["不换气?","有脚蹼?"]

In [75]: res = classify(myTree,labels,[u'是',u'否'])

In [76]: print res
不属于
```
__存储决策树:__

```python
def storeTree(inTree,filename):
    """序列化存储决策树
    """
    import pickle

    with open(filename,'w') as fw:
        pickle.dump(inTree,fw)
```
__加载决策树:__

```python
def grabTree(filename):
    """加载决策树
    """
    import pickle
    
    with open(filename,'r') as fr:
        tree = pickle.load(fr)

    return tree
```

## 示例:

讲解决策树如何预测患者需要佩戴的隐形眼镜类型

## 总结:
* __优点:__ 计算复杂不高,输出结果易于理解，对中间值的缺失不敏感，可以处理不相关特 征数据。 
* __缺点:__ 可能会产生过度匹配。
* __适用数据类型:__ 数值型和标称型。