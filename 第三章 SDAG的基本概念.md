# SDAG基础

## 概述

对父单元的引用是建立单元排序的基础，这些引用关系同时也生成了整个区块链结构。由于我们尽量的引用多个父单元，因此可以快速的收敛并发的情况。每个单元都可能会有多个的父单元和多个子单元。我们沿着父子关系在历史上的演进可以观察到，当一个单元被多个新产生的单元引用时就会产生分叉，而当一个单元引用多个早期单元时又会看到收敛。这种结构在图论中称为有向无环图或DAG。

在SDAG图里，有些单元是由公证人创建的**公证单元**，有些则是由用户创建的**普通单元**。每个普通单元都包括了若干条事件记录。不同于传统区块链技术的链式结构，SDAG采用图的结构来保存各种事件。这些事件按照特定的共识算法形成了一种全局顺序，也就是说，每个事件都可以计严格计算出唯一的全局逻辑时序，任何两个不同的事件我们都可以通过全局序来判定他们发生的先后顺序。

在SDAG网络中，参与的主体由三种角色构成，HUB，公证人和客户端。他们可以向SDAG网络并发的独立的提交签名单元。形成的单元之间相互验证，相互确认，随着数据的不断推进，所有发生的历史越来越难以篡改。不同于比特币，SDAG中不存在算力攻击，攻击者不可能改变其他人已经确认的历史事实。


## 单元

在有向无环图中的每一个节点，我们称之为一个**单元**。有向无环图中的每个边，我们都称之为一次**引用**，它只能有一个方向，指向一个比它更早生成的**父单元**。每一次引用都会形成对之前历史的一次确认。那些验证失败的单元不会被引用和广播，因此最终也不会被包含到有向无环图中。SDAG中第一个被创建出来的单元称为**创世单元**，它是由所有的公证人共同签名产生的，通常用字母G来表示。从图中任意一个单元出发，沿着父单元向过去回溯，最终都会到达创世单元。一个单元选择的父单元一定是经过验证的单元，验证失败的单元无法成为父单元。如果一个单元选择了一个验证失败的单元作为父单元，那这个单元本身也不能通过其他人的验证。一个单元允许选择多个父单元，目前我们允许的最大值为16个父单元，选择的这些父单元必须是不同的作者，且这些父单元之间不能相互包含。

如果一个单元A沿着父亲方向回溯能够到达另外一个单元B，我们称单元A包含单元B。显然，所有的单元均包含创世单元。单元A包含单元B，单元A引用单元B，单元B是单元A的祖先，单元A是单元B的子孙，这些表述都是相同的意思。

那些没有被任何人引用的单元，我们称之为**顶点单元**。所有的顶点单元共同决定了一个有向无环图的形状。一个单元引用多个父单元，本质上是对这些父单元所代表的历史的一次总结，从这个意义上来说，所有的顶点单元包含了这个有向无环图的全部历史信息。

一个单元包含了以下的**基本信息**：

* 单元的作者和签名
* 选择的父单元的列表
* 创建该单元时所看到的当前稳定主链单元
* 签名的事件列表
* 版本信息及其他

除了这些基本信息以外，单元还会推导出以下一些**属性信息**（具体参考下面的名词解释）：

* Level
* 见证级别 WL
* 最小见证级别 MIN_WL
* 最优父单元 BP
* 所属的主链序 MCI
* 最近的包含主链序 LIMCI
* 在所属主链序中的子序 SUB_MCI
* 最终稳定后的哈希 BALL

当用户向SDAG增加数据时，他会创建一个相应的单元并广播扩散给其他节点。我们只需要简单的广播单元的基本信息即可，不需要挖矿，不需要许可，随时发送。当新增的单元很少时，SDAG图看起来很像一个链，只有很少的分叉并很快收敛。当大量并发单元同时上链时，SDAG图看起来就会变的复杂起来，不停的发散，又不停的收敛。

就像在传统区块链一样，每个新的区块都可以确认之前的所有区块（包括区块中的交易），SDAG中的每个单元都会确认他的所有祖先单元。如果一个人尝试修改一个单元，必然的要修改单元的哈希值，这势必会破坏引用该单元的所有子孙单元的的签名和哈希值，因为他们都依赖于该单元的哈希。因此，如果不能跟所有子孙单元协作或者直接偷取他们的密钥，修改一个单元是不可能的。一旦一个单元广播到网络上，并且其他用户开始在这个单元上构建自己的单元，把他作为祖先单元，随着图形的推进和单元的不断确认，修改这个单元所要付出的代价就会无限增长。在SDAG中对单元的引用是一种互助行为，用户构建一个单元的同时，也确认了它先前的单元。我为人人，人人为我。

## HUB

HUB负责将客户端的签名交易打包，收敛当前的顶点单元，形成一个新的单元，并且将这个新单元向全网广播。HUB也会验证其他人通过网络发送过来的单元。所有的hub形成了SDAG的网络的核心。公证人发送的公证单元会通过连接的Hub进行广播，普通客户端会将自己的签名数据发向hub进行打包广播，同时还可以通过HUB来查询图形的所有相关信息，比如历史交易，账户余额等。HUB在本地保存了SDAG的全部历史记录。每个HUB都会计算自己看到的主链推进情况，从而在本地确定事件的发生顺序，正确的更新当前的业务状态。

## 公证人

SDAG网络的参与者中，有一些是长期建立了良好声誉，并且对维持网络健康利益攸关的组织或个人，我们称他们为**公证人**。公证人负责对接收到的未稳定普通单元进行收敛确认，并发送由自己签名的公证单元。公证单元并不包含任何业务信息，它仅仅是对先前历史的确认。公证人是一种特殊的客户端，他们保存了全部的图形信息。外界除了知道公证人的地址以外，无法确定跟踪公证人的实际网络位置，因此也进一步增强了网络的整体安全性。每个公证人会对未稳定的普通单元进行独立的验证，并且发送公证单元确认它，直到他观察到SDAG图中已经没有未稳定的普通单元。

虽然默认他们的诚实的，但只信任其中一个或部分公证人也是不合理的。我们期望他们诚实的发布公证单元，那么多数公证人都认可的历史就应该认为是真实的。所谓诚实地发布公证单元，就只有一个简单的规则，即公证人创建的公证单元必须严格顺序的包含自己上一次创建的单元。如果同一个作者发送的单元不包含自己之前发送的单元，这种行为是不被认可的，我称之为**非连续**。我们允许不超过三分之一的公证人发送非连续单元，只要系统有超过三分之二的公证人诚实公证，我们所计算的主链就可以达成共识。

## 客户端

客户端并不保存全部的图形信息，只是向hub发送由自己签名的事件数据。并且从hub获取可以验证的数据。客户端不必要保存SDAG所有的数据到本地数据库中，因此可以应用到网页APP中和嵌入式设备当中。

## 主链

SDAG共识的核心算法是推进主链。**主链**是SDAG有向无环图中按照特定规则形成的一条特殊路径。我们总能在图中找到一条包含了绝大多数公证人所创建的公证单元的主链。所有的公证人共同推动了主链的持续推进。每个单元最终都会被主链所包含，并最终**稳定**。稳定之后的单元可以确定它的唯一的全局序。除了单元的基本信息以外，稳定单元还会计算出一系列的属性值，包括MCI，LMCI，事件发生时的状态，以及一个对所有这些历史信息的哈希值（相当于当前单元视角下默克尔树的根）。而主链的计算是和公证人发送的公证单元相关的。公证人不断的发送公证单元，从而可以使得主链持续向前推进，最终使得所有的普通单元都达到稳定。有关主链推进的详细描述参见第七章。

## 名词解释
单元的属性意义和计算方法如下, 用C来代表当前单元，P来代表父单元

### Level
Level代表单元到创世单元的最远距离。Level是图形上事件发生先后的一种逻辑时序。创世单元的Level为0。显然，一个单元的Level取决于它的最大Level的父单元。
```
C.level = Max(P.level) + 1
```

### BP
最优父单元表示所有父单元中包含最新大多数公证人的单元。比较两个父单元优先级的算法如下：
首先选择WL较大的单元；如果WL相同，则优先选择公证单元；如果不能确定，则选择Level较小的单元；如果还是无法区分，则选择哈希值小的单元。


### WL
见证级别：从当前单元开始沿着最优父单元方向回溯，直到遇到绝大多数的公证人，定义此时的公证单元为相对稳定单元**RP**，它的Level就是当前单元的见证级别。

```
C.wl = RP.level
```

### MCI
MCI是主链序Main Chain Index的缩写。主链是图中一条包含了大多数公证单元的路径，可以一直回溯到创世单元。创世单元的MCI为0，新的主链单元MCI依次增加1。被当前主链单元包含且不被上一个主链单元包含的所有单元均属于同一个逻辑时间区间，他们拥有相同的MCI。

```
如果C也在主链上，则：
C.mci = BP.mci + 1
```

### LIMCI
LIMCI表示当前单元所包含的最近主链单元的MCI，也就是所包含的主链单元中MCI的最大值。

```
C.limic = Max(P.limci)
```

### MIN_WL
最小见证级别是一个推主链时使用的指标，它的计算方式如下

```
C.min_wl = RP.wl
```

### SUB_MCI
在相同的MCI的所有单元中，按照他们图形上发生的先后顺序排列所形成的序号。其中Level越小的单元SUB_MCI越小，Level相同的，单元哈希越小的SUB_MCI越小，可以看到，主链单元是SUB_MCI最大的，因为它包含了所有MCI相同的其他单元。

### BALL
当一个单元稳定之后，我们可以确定它所包含的事件发生后的业务状态，所有父单元此时均已稳定。由此我们可以计算一个包括其属性，状态以及父单元的最终哈希值。这个哈希值相当于对过去历史的一个摘要，类似于默克尔树的根。它是可以验证的，如果同一个单元两个节点计算出来的BALL值不相同，那么就说明它们之间没有达成共识。

### LAST_BALL
为了保证单元构建的正确性和可验证性，我们需要每个单元包含其构建当时视角下最新的主链稳定单元以及对应的BALL值，这个当时的最新主链稳定单元，也叫稳定点，他就是当前单元的**LAST_BALL**。LAST_BALL不能比任何父单元的LAST_BALL更早（MCI更小），也就是说必须保证LAST_BALL要么沿着主链向前推进（MCI增加），要么不动（MCI不变），但不能向过去后退（MCI减小）。

```
C.LastBall.mci >= Max(P.LastBall.mci)
```

LAST_BALL是用户签名保护的一部分。在一个节点验证单元时，它会验证这个稳定主链单元的BALL值是否和自己的计算结果一致，如果不一致，说明共识失败了，这个单元也会被丢弃。

## 基本推论

* 通过以上定义，我们不难得出以下推论:

```
1. P.level < C.level
2. P.wl <= C.wl
3. P.mci <= C.mci
4. P.limci <= C.limci
```

* 如果单元A和单元B满足：
```
A.level >= B.level 或者
A.wl > B.wl 或者
A.mci > B.mci 或者
A.limci > B.limci 或者
```
则单元A不包含单元B