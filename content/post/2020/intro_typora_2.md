---
title: "【十八般兵器】Typora进阶使用-画图篇（1）"
date: 2020-10-15T17:03:23+08:00
categories: ['十八般兵器']
draft: false
---


![banner](/img/2020/intro_typora_2/header.png)



Typora除了能应用在日常的写作中以外，还有一项强大的功能，就是在文章中插入各种炫酷的图形。这里说明一下，Typora的画图功能，其实是基于markdown语法的mermaid的功能，具体相关信息可以[参考官网](https://mermaid-js.github.io/mermaid/#/README)。



# 1. 什么是Mermaid？

​		简单来说，**mermaid就是一种用文本形式来画图的工具，其应用场景就是在markdown语法环境中，绘制一些简单常用的图形**，例如流程图、状态图、甘特图等，这种语法是基于JavaScript环境的绘图工具，因此可以在大多数的markdown编辑器中使用，今天我们就来介绍如何在typora中来使用Mermaid。

# 2. 如何在typora中插入Mermaid图形

​		在typora中使用Mermaid画图非常简单，我们只需要在文本框中输入````Mermarid` 即可自动插入一张新的空白图画，我们就可以开启美妙的typora绘图之旅了。😁

![image-20210115172603784](/img/2020/intro_typora_2/image-20210115172603784.png)



# 3.支持的图类型

## 3.1 流程图-Flowchart



作为工作中最常用的一类图形，流程图在描述事物过程方面非常直观。

在typora中绘画流程图的基础语法如下：

```markdown
graph LR
	node1-->id2(node2)-->id3([node3])
```

![image-20210115190002905](/img/2020/intro_typora_2/image-20210115190002905.png)

语法说明：

- graph LR： 这一行说明要绘制的图形的方向。
  - LR：从左到右，left to right
  - RL：从右到左，right to left
  - TB：从上到下，top to bottom
  - BT：从下到上，bottom to top

- 定义节点：有两种定义节点的方式
  - 可以直接输入文字信息作为节点名称: node1
  - 也可以在节点名称前加一个id的形式：id2(node2)，这样后续可以通过id1来引用node1这个节点。
  
- 节点图形形状：

  - 默认：方形 
  - （node2）: 圆角
  - ([node3]) ：椭圆

- 条件节点：在绘制流程图中会经常用到条件节点，在Mermaid中也可以实现的。

  ```markdown
  graph TD
      A[买家收货] --> B{有质量问题?}; #{}可以画出菱形，表示条件节点
      B -->|Yes| C[发起退货];
      C -->D(卖家退款);
      D --> F
      B ---->|No| E[确认收货];
      E --> F(流程结束)
  ```

  
  
  ![image-20210115190419646](/img/2020/intro_typora_2/image-20210115190419646.png)

## 3.2 时序图-SequenceDiagram

时序图是在设计多个对象之间信息交互时常用到的图形，typora+Mermaid也可以轻松实现的。

```markdown
sequenceDiagram ## 定义时序图
    participant 合法用户A  # 定义参与者
    participant 非法用户B
    participant 后台系统
    合法用户A->>后台系统: 发起登录请求 #定义信息交互过程
    后台系统->>合法用户A: 验证通过，登录成功！
    非法用户B->>后台系统: 发起登录请求
    后台系统->>非法用户B: 验证失败，拒绝登录！
```

![image-20210115190502010](/img/2020/intro_typora_2/image-20210115190502010.png)



# 4.总结

这里简单介绍了两种日常工作中使用率较高的图形类型，这两种类型以及基本可以应付日常工作中80%以上的画图需求了，后面有时间再总结一下其他经常用到的图形，例如甘特图，实体ER图等。👋



