# FitBlock:使用纯Node.JS实现的点对点电子现金系统
本文提出通过Node.js实现区块链系统，并可以实现Web在线挖矿。目前系统基本开发完成，将处于测试阶段和最后准备阶段(包括多机可用性测试，文档编写，准备上帝区块，部署引导节点)。而区块链对于大多数开发是神秘的，尤其语言鄙视链的存在，让其它人对于JS开发区块链系统有一定偏见。所以我希望能通过本人开发区块链的一些经历把区块链的技术要点做一些简单的分享。


本文开发区块链源码地址：[FitBlock仓库地址](https://github.com/FitBlock/fit-block)

# 剧前提要
SHA-256是一种摘要算法，和大家熟悉的MD5算法类似，就是将大量或少量数据来生成几乎唯一的一个极大数。但SHA-256长度偏长且摘要算法更加安全。


ecdsa-secp256k1是一种椭圆加密算法，可以将私钥推导出公钥。且通过私钥加密，公钥可以验证签名是否正确，而且几乎无法通过公钥推算出私钥的算法，这里亦使用了纯JS来实现算法([ecdsa-secp256k1仓库地址](https://github.com/zy445566/ecdsa-secp256k1))。

# 工作量证明
何为工作量证明，比如在很多公司喜欢粗暴地将工作时间和工作量挂钩，催生了996文化，狗性文化。这属于管理者的一种懒政，或者说通过简单的或低成本方式来验证员工的工作量，至少不用24小时盯着你看。那么在程序中，如果计算机是一个只会做运算的工人，如何设计一种算法来通过低成本的方式来计算计算机计算的工作量呢？


其实答案很简单，举例我需要你在0到1OOO亿中，使用摘要算法比如SHA-256找出开头是888888的摘要字符串的那个数字，那么你可能找了好几百万次才找到这个数字，这个数字计算出的摘要字符串开头是888888。但是我要校验就很简单，只需要将你给我的这个数字，我用相同的摘要算法进行运算得到摘要字符串之后，验证它的前6位是否是8就可以找到了。


工作量证明只是方法，存在问题就是如何在计算机算力高速发展或矿工大量罢工导致算力突降的基础上实现工作量的稳定动态增加或动态降低。这点其实很简单比如你计算6个8的摘要字符串的时间大于十分钟，则我把难度调成下个摘要只需要计算5位，如果在十分钟内完成，我把下个摘要增加到计算7位。

# 钱包和交易
在现实的理想的世界，你的钱包只有你一个人才能打开，并只有你想要买东西的时候，才会打开钱包为你想买的东西付账。那么在网络世界应该如何实现一个类似钱包的东西呢？


在剧前提要提到过ecdsa-secp256k1椭圆加密算法，这种算法可以使用私钥进行签名，且公钥可以验证签名，且公钥难以反推出私钥。那么我们是不是可以使用私钥作为我们的钱包的钥匙，当我们需要付钱的时候使用私钥进行签名，当别人需要确认交易的时候，只需要将公钥验证交易的签名是否通过，那么这个交易就是经过你授权且有效的。


那么私钥其实是做了一个取钱的钥匙，那么存钱的入口在哪里？ecdsa-secp256k1椭圆加密算法有一个重要的特点就是私钥可以推导出公钥，那么公钥是不是就能作为钱包地址即存钱的入口。只要你能通过私钥推导出这个公钥，就证明这个钱包是你的。


# 区块链
那么上面说了工作量证明和交易，那么工作量证明的意义体现在哪里？需要通过什么东西来承载这些交易数据。对！就是区块链。


什么是区块链？讲区块链前，我认为先讲单独的区块是什么可能更为合理，单独区块可以理解为使用算力的工作量为担保，来证明交易是否真实的证据。那么区块链就很好理解了，通过区块链把这些区块串联起来。

那么区块是如何串联的呢？在FitBlock是这样做的，比如第一个区块链通过工作量证明计算出了一个888888...666666即开头是888888结尾是666666的一个摘要字符串，假设它的高度是0，那么下一个区块则要计算开头是666666的区块，这个时候它的高度是1，以此类推那么下一个区块则需要再取上个区块摘要结尾的N位来计算下一个区块，则下个区块的高度继续加1。


那么根据上面的一点，你要推翻一个区块，则需要找到计算出和它一样开头的区块摘要，但是假设目前区块高度是999，而你要推翻第1个区块，那么你要从第一个区块计算到第1000个区块，只有你的区块高度大于别人的区块链才表面你有足够的证据推翻，但这个几乎是不可能的，因为每个区块都是根据当前的区块链上算力来推算出，即使你要推翻第高度是999的区块，都要使用当前区块链上51%以上的算力才能推翻上一块。只要所有正义者算力大于51%，坏人都无法得逞。


而区块链几乎是无法被篡改，那么你的交易只要接受被矿工接受，并打入区块，你的交易就几乎是无法被篡改的？那么问题来了矿工为什么要为你打包交易进入区块？


其实在一定高度区块链的区块链中，在某个矿工进行工作证明后出块是有出块奖励。并且你的交易都会有一定交易费给予矿工，保证了矿工的收入。

# P2P网络
那么在区块链中传播是一个十分重要的环节，要能即时发现节点并在周围节点启动时能够及时发现节点，包括在内外网络的节点打通问题，该如何处理.


我认为首先需要一个引导节点尤其是世界上节点较少的时候，引导节点可以使连入的节点快速发现其他节点，通过节点和节点之间的节点分享，快速建立节点网络。其次通过引导节点来引导内网节点来做节点中继，帮助内网也能快速连接到外网的其它节点。


其次需要保证某个节点只负责一定的范围，这样只要周边节点上线就可以以最快的速度发现节点，同时优先周边节点本身网络距离近，那么网络传输必然比远程节点更加迅速。减少网络的传播时间。


# 结语
第一次使用JS实现一个完整的区块链系统。也算是遇到了一些坑，在下次实现区块链系统时，不要以复杂的系统为考虑，提前把系统做的复杂。第二个坑是使用了GRPC，因为GRPC有Node.js模块是使用了C++模块，导致我要前后端使用一个核心库，要重新进行分离。第三过早的使用Web Components好处特别轻量,周边过于不完善，对于CSS弱没有UI库确实比较麻烦。在写这个系统的的时候，我认为最佳部分是使用了TS，在TS的加持下，确实很多东西方便了许多。其次使用lerna和git的submodule管理不同模块，管理和提交都相对清晰。