# SDAG特点

SDAG是基于有向无环图技术全新设计的高性能分布式共识系统，支持高并发，秒级确认，无需挖矿，增加单元无需等待。SDAG立足于下一代区块链技术，彻底解决了传统区块链的扩容问题和回退问题。

SDAG的每个单元通过引用之前的一个或多个单元的哈希值相互关联，多个单元之间的引用形成一个有向无环图。在验证单元的同时，建立起他们的全局顺序关系。系统中没有一个中心化的实体负责管理或协调单元的进入，每个用户只要按照规则构建单元并且对其签名，就可以向SDAG网络中广播新单元。 随着新单元的不断加入，早期单元会收到越来越多的后续单元直接或间接的确认。

SDAG本身是一套防篡改去中心化共识系统，可以存储任意事件。由于建立了任意事件的全局序共识，因此可以基于此开发任何可能的业务，比如发行资产，交易，存证和溯源等，包括体现价值转移的数据，例如货币、产权、负债、股权等。其模块化设计方便用户像搭积木一样搭建自己的区块链业务系统。SDAG插件丰富，拥有完备的工具集，可以对各行各业的应用场景进行广泛地支持。

全局序通过在有向无环图中计算出一条主链来建立，主链是由被称为公证人的用户所创建的公证单元推导出来的。一个单元被主链越早包含，则认为他的全局序越早。用户增加一个单元后，随着后续公证单元的不断确认，这个单元最终会被最新推进的主链确认，一旦被主链确认，这个单元就会达到稳定，并且获取全局序，这是一个确定性的过程，没有概率性的回退问题。

传统的区块链是链式结构的，生成一个区块有严格的时间限制，由于链式结构，导致很多生成的区块无法并发的增加到系统中，而且基于POW的共识会造成极大的能源浪费，当发生网络分区时，还会造成概率性的数据回退。基于有向无环图的共识算法解决了这些问题。任何用户创建的单元可以并发进入系统，没有时间限制，满足实时上链，没有昂贵的挖矿过程，并且共识是确定性的。

SDAG采用RUST语言开发，采用最新的协程技术和无锁数据结构优雅的解决了高并发的难题，可以使系统资源达到最大化的利用。同时RUST语言的高性能和安全性，也保证了SDAG整体的稳定性和安全性。

