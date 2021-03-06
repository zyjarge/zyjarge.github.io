---
title: "【三省吾身】互联网企业数据工程中的那些坑（二）"
date: 2018-05-11T15:14:25+08:00
draft: false
categories: ["三省吾身"]
---

​	继续上篇文章继续谈谈数据工程中常见的一些问题，以及自己的浅显间接。

### Q3：由于指标统计口径不一致，导致数据的可信度下降。

A3：这个问题其实可以引申出另一个问题，**为什么用户总觉得数据不够准确？**

​		首先，在处理数据需求的时候，在需求开发阶段，就需要与业务明确好该需求的业务背景和逻辑是什么，数据来源是哪里，以及该需求的目标用户是谁，因为及时是同一个需求，不同角色所关注的数据往往是不同的，老板一般肯定不会关注日常的运营细节数据吧。

​		在明确了上面这些信息之后，需要把这些信息做好需求备案工作，尤其是在需求讨论过程中反复确认的细节，还有那些TODO相关的内容，做好了这些工作后，还需要验证一下数据质量，一般可以抽样看下实际的数据，与预期是否一致，如发现不一致一定要提前做好应对方案，以防止在后期对数据的准确性造成影响。



### Q4：业务方经常反映数据部门出的数据不够及时。

A4：首先判断瓶颈在哪里，一般来说可能是如下两个方面：

- **资源瓶颈：** 需要对数据平台的资源能力与业务量的大小有个底，即确定目前的资源与目前的业务流量是否匹配，一般情况下，对于目前的业务能力来说，**计算资源和存储资源占到整体的70~80%为宜**，低于此水平，说明资源没有被充分利用起来，超过该水平，就需要进行扩容操作了。
- **调度存在优化空间：** 计算任务的调度作业必须是最优的，这里所说的调度作业最优是指：首先，计算任务执行时间安排与需求的优先级是吻合的，即执行按照从先到后，优先级按照从高到低的顺序进行排序。其次，尽量避免高峰期的任务扎堆执行，看看能否将部分高负载的任务错峰执行。再次，如果上述两个方法仍无法解决，则需要从业务角度入手，看下目前的任务调度优先级与实际的业务需求是否匹配，避免将优先级不高的任务排在最前面执行，将更宝贵的资源留出来给价值更高的业务作业来执行。
- 最后，任何数据任务都要有生命周期的概念，即计算任务不能无止境的存在系统中，必须避免业务已经下线许久，但其对应的数据需求仍然占用计算资源和存储资源的情况发生。



### Q5：遇到数据出现异常的情况，如何快速定位问题，以及快速解决。

A5：一般来说，数据异常的问题需要根据不同的问题症状，做不同的分析和操作。

- **症状1：没有数据产生：** 这种情况一般需要从数据源入手排查，查看数据源头是否正常，例如根本没有接收到数据或者由于存储满了等原因，如果没有问题的话，再看看对应的任务执行是否成功，一般任务失败后重跑下任务即可解决。

- **症状2：出现数据骤增和骤减的现象：** 这种情况下，一般可以排除按如下策略进行排查：

  - 首先还是从源头进行排查，看看是不是原始数据源本身就存在掉量的情况，可以对比一下环比和同比的数据；
  - 其次，看看近期是否有过线上升级操作，包括调整过计算逻辑，上报日志发生变更等情况，之前在实际工作就遇到过因为发布新版本后，部分机型出现闪退的现象，导致异常情况；然后也可以找下运维的同学了解一下，网络底层近期是否有过异常情况，例如机房网络故障等。

- 总体来说，解决这类问题的思路，就是先看整体的变化情况的特征，例如增减的比例大小，然后再把整体数据进行细分，一般可以拆分的维度可以是：不同平台（移动端、PC端、Pad端，网页端）、不同地域、不同版本、不同渠道、不同日期等等。把整体数据按照不同的维度进行切分，再看每个分量的变化特征是否与整体相似，一般如果细分之后的变化特征与整体相似的话，就可断定是该维度的异常所导致的。

  

### Q6：业务方总反馈，提的数据需求响应不及时，需求上线经常延期，直接导致影响运营效果。

A6：经常碰到业务方的同事向我反映，给你们提了某个数据需求，为什么都过了好久都还没有上线，在业务方的眼中好像数据研发的效率很低，但实际上研发这边也是一肚子苦水，感觉永远做不完的需求，刚刚做完的需求又经常改来改去。这个问题似乎是双方一直都无法调和的矛盾，但只要稍加分析便可看出些端倪。让我们简单分析一下，时间都花费在了哪些地方。

- **需求沟通上：** 业务方提的一般都是原始需求，即不能直接输入到研发端进行开发，需要数据产品经理在中间进行需求确认和梳理，在整个这个过程中，所需要的时间主要与如下几个因素相关：
  - 业务人员对业务的熟悉程度
  - 业务人员的数据思维能力
  - 产品经理和研发对业务的熟悉程度，即理解业务人员的需求的接受程度
  - 是否具有恰当的沟通机制，即能否做到高效的充分沟通，即面对面沟通 > 电话沟通 > IM、邮件沟通 > 不沟通，
- **研发阶段：** 好容易需求沟通完成了，进入开发阶段，对于比较复杂的需求，作为研发来说，需要从数据ODS层开发，再往上一层一层的建模，这个阶段主要与如下几个因素相关：
  - 研发的业务能力，即研发能否根据业务方的需求，给出恰当的解决方案。
  - 数据平台架构的质量，能否很轻易的加入新的需求，而不会影响已有的代码逻辑
  - 是否具有良好自动化的测试架构，保证代码质量
- **上线阶段：** 需求都开发成功之后，上线交付是否顺利也是至关重要的环节，一般来说与如下因素有关：
  - 是否有严格的发布流程规范
  - 是否有持续集+自动部署+自动验收机制
  - 与业务方的交付方式，一般小的需求通过发送邮件告知，大的需求则需要召开专门的需求交付会，进行沟通和培训需求方使用。

### 结束语：

​	花了几天时间，把以往工作中遇到过的具有典型性的问题简单梳理了一下，并给出了自己的想法，当然这些肯定并不完美的最优解，待后续有更加完美的解决方案后再来更新。另外由于时间精力有限，有很多问题没有记录到，只选取了其中比较有代表性的几个问题，后面如果遇到新的问题，也会再写几篇类似的文章。

