---
title: "【三省吾身】互联网企业数据工程中的那些坑（一）"
date: 2018-05-02T18:19:30+08:00
draft: false
categories: ["三省吾身"]
---

 

​		在互联网的公司的数据岗位大摸爬滚打了10多年，企业内部数据工程实践过程中也遇到过各种各样的坑，有的坑更是反复遇到，有的问题是一次性的，而有的问题又是长期存在的，想着本着【吃一堑，长一智】的原则，想着把自己遇到过的问题以及对应的解决方案（有的问题仍然未解决）做成一个专题，于是本篇文章就是该专题的第一贴。

### Q1：如何解决临时取数需求占用数仓研发人员大量时间这个问题？

A1：理论上来说，数仓的工作人员只负责开发层面的事务，具体到业务层面的数据提取工作，则是业务人员自身的工作。

​		我们不妨深入分析一下出现这种问题的深层原因：

- 首先，是否已经搭建了易于使用的自助取数工具？由于业务人员一般都不具有编写SQL的能力，因此这里说的取数工具应该是傻瓜式的，类似Redash、superset、 metabase等这种可视化工具，并且在搭建成功以后，务必向业务人员进行了相应的培训。
- 其次，从需求角度入手，因为有的数据需求并非是常规的业务需求（例如boss突发奇想的需求），这中情况下一般确实无法从现成的应用层中取数，一般都需要到中间DW层甚至底下ODS层去取数，这种情况确实需要数仓的研发人员介入。**但是，这里有个小tips，作为研发一定要反向追踪一下该需求的背后业务逻辑。**因为这种需求一般有第一次就会有第二次，如果提的次数多了，可以考虑做成一个固定的报告。
- 最后，还有看下花费的大量时间，是实际消耗在哪个环节，例如是否浪费在业务需求的沟通过程中。



### Q2： 数据的跨部门合作的问题？

A2：数据作为企业业务中的“血液”，几乎贯穿了各个业务线，因此有时候必须面对跨部门合作的问题： 

+ 由于数据工程具有一定的专业性，业务人员可能对数据工程不太了解，但研发同学一般又只了解技术，因此会出现不同业务部门所站角度不同，往往都对自己的一亩三分地很熟悉，其他部门的工作不太了解的问题。

- 跨部门会涉及到角色不同而参与程度不同的问题，具体可参考敏捷工程中“猪”和“鸡”的故事。

  我是这么看这个问题的，一般涉及到跨部门合作的时候，双方需要做到在一定程度上了解对方的业务逻辑和特征，需要了解对方部门是以什么角色跟本部门合作的。

  - 对方是数据生产者：
    - **信息确定：**需要了解对方的数据产生的形式，是数据库，数据接口，日志文件等，以及还需要了解对方的数据体量大小，会涉及到数据资源的规划和调度问题。
    - **元数据管理：**元数据管理也很重要，对于每个字段都要有明确的schema定义，以及schema后续升级的方案等，因为之前遇到过对方接口升级之后，没有及时通知接口使用方导致数据出错。
  - 对方是数据消费者：
    - 一般这种情况是直接对接业务方，需要了解对方的业务逻辑背景，一般来说业务人员提的数据需求较为模糊，无法准确的用标准的数据思维来思考，或者用数仓的语言来描述自己的需求。**这时数据产品经理的重要性就体现出来了，Ta即需要了解业务，也需要了解技术，做到既能听懂业务语言，有能听懂研发的语言，做好业务和研发人员的“翻译”。**
  - 跨部门之间一定达成统一认识：
    - 俗话说兄弟齐心，其利断金，这句话放在这里一样适用，数据部门也业务部门就是一对兄弟，脱离谁都无法单方面存在。但实际上知易行难，这和很多其他因素有关，例如组织结构、公司内部是否注重数据驱动等等，这个话题可以改天单聊一下。
