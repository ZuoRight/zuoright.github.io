# Bitcoin

比特币是最早被创造出来的去中心化加密货币，诞生于 2008 年世界金融危机之时

- 2008.8.18 `Satoshi Nakamoto`(中本聪) 和 `Martti Malmi`(马尔·蒂马尔米) 注册了域名：[bitcoin.org](https://bitcoin.org)（准官网）
- 2008.11.1 中本聪发表了比特币的白皮书：《Bitcoin: A Peer-to-Peer Electronic Cash System》（比特币：一种点对点的电子现金系统）
- 2008.11.9 比特币项目在 sourceforge.net 平台注册
- 2009.1.3 中本聪打包出了创始区块（`Genesis Block`）

![20220718221933](http://image.zuoright.com/20220718221933.png)

> 创始区块中附带了一段文本：  
> The Times 03/Jan/2009 Chancellor on brink of second bailout for banks  
> 《泰晤士报》 2009年1月3日 英国财政大臣正欲对银行业实施第二轮救助

![20220718221945](http://image.zuoright.com/20220718221945.png)

- 2009.1.9 中本聪在 sourceforge.net 发布了比特币`0.01`版本源码
- 2010.5.22 拉斯诺用挖到的比特币购买了两个比萨，共花费10000BTC，这是比特币第一次被用于实物支付
- 2010.7 世界上第一家比特币交易所 Mt.Gox(门头沟) 在日本东京成立
- 2010.12.12 中本聪在 bitcointalk.org 论坛发布了比特币`0.3.19`版本公告，然后便销声匿迹，此时比特币价格约`$0.25`
- 2014.3.7 中本聪在 p2pfoundation 论坛发布了一条辟谣消息：我不是多里安·中本聪，此后便再也没出现过

## 区块

比特币的区块链可以看作是运行在P2P网络中的公开账本，一笔笔交易存储在区块结构体中，然后被打包成一个区块，一个个区块按更新顺序串联成一条区块链，第一个区块被称之为创世区块。

打包区块的过程叫做工作量证明(PoW)，因为可以从中获取奖励，所以俗称挖矿，由分布式网络中的节点来完成，俗称矿工，用户购买硬件成为矿工。

> 区块奖励最初为50个BTC，每开采 210,000 个区块则会减半，2012年首次减半到25个，2016年减半到12.5个，2020年5月减半到了6.25个，平均每四年一次

![20220721193416](http://image.zuoright.com/20220721193416.png)

每个区块由区块头和区块体两部分组成

![20231027211035](https://image.zuoright.com/20231027211035.png)

> 示例参考：<https://andersbrownworth.com/blockchain/public-private-keys/blockchain>

交易按顺序存储在区块体中，比特币诞生之初，单个区块最大可一到32M，但由于有人恶意制造的大量小额转账使网络中有大量的待确认交易，影响了网络运转，所以改为了1M（大约1500～2000笔交易）

第一条交易是矿工写给自己的挖矿奖励，称作Coinbase交易，然后才是普通用户的交易。

当然交易可以只有Coinbase（即空块，比如创世区块），但通常矿工不会这样做，因为打包其他交易还可以赚取手续费。

矿工在写奖励时不能乱写，否则会验证失败，白费精力，不过矿工可以优先选择打包手续费高的交易

矿工每挖取并成功更新一个区块，获取的激励为：`区块奖励`+`交易手续费`，正因如此，才会有人愿意主动去做这件事，比特币网络才得以正常运行。

但挖矿并没有那么容易，需要满足一定的条件

首先将每一笔交易哈希，然后每两个交易的哈希再哈希，依次递归，如果遇到奇数则使用自身的副本配对，最终会形成一颗倒立的哈希二叉树，其根哈希被称作默克尔根(Merkle)。

> 哈希二叉树，是一种用作快速归纳和校验大规模数据完整性的数据结构，在比特币网络中，可以通过默克尔树快速地校验某个交易是否存在于某个区块中。

默克尔根与父区块的哈希等组成区块头

- 父区块的哈希
- 默克尔根
- Nonce（随机数）
- 其它：区块高度、bits(难度系数)、时间戳等

![20220721173614](http://image.zuoright.com/20220721173614.png)

然后将区块头哈希，如果生成的哈希值小于某个目标值，便可以广播到区块链网络，反之则需要重新哈希。

目标值是基于难度系数根据公式动态算出来的，由256位二进制数组成，每一位不是0就是1，0的概率为1/2，要求小于目标值，实际就是要求前n位有多少个0，即概率为(1/2)的n次幂，理论上目标值越小则需要0越多，概率越低，挖矿的难度越大。

观察区块头可知，除了Nonce和时间戳外其它都是不能随意改变的，其中时间戳并不需要时时改变，只要大于上一个区块的时间戳并且在将来2小时以内即可。所以通常只能用穷举法不断地调换随机数来满足目标条件，这个过程便称之为工作量证明(Proof of Work)，简称PoW。

> 不同矿工打包时的Coinbase交易收取地址不同，而且打包的交易也不一定相同，所以每个矿工要寻找的随机数也是不同的。

比特币网络的算力和难度一直呈上升趋势，目前全网算力大概200EH/s左右，而单机CPU算力不过几M，GPU算力也不过1G，所以单机挖矿的成功率几乎等于0，现在主要是专用的ASIC芯片构建的矿池挖矿。

> 算力：每秒产生哈希碰撞的能力，最小单位：H(Hash)，即1次运算  
> 1`E`=1000`P`=1000,000T=1000,000,000`G`=1000,000,000,000`K`=1000,000,000`G`=1000,000,000,000,000`H`

平均10分钟左右挖出一个区块，1小时就是6个，约每4年奖励减半一次，大致可以推算出比特币的总量约为2100万个，预计在2140年全部挖出。

> `6x24x365x4`x`50(1+1/2+1/4+1/8+...)`=`2100W`

## Reduction & Halving Countdown

> <https://www.oklink.com/halving>

计算公式：`(Halving/Reduction block – Next block height) × Average block time – Estimated time for the next`

![BTC Halving Countdown](https://image.zuoright.com/BTC%20Halving%20Countdown.png)

## Difficulty Adjustment Period

为了保证生成一个区块的时间恒定，挖矿难度每过一个固定周期（2016个区块）就会调整一次。

计算一下上一个周期的平均耗时，理想时间为两周（1,209,600秒，平均10分钟一个区块），如果过快则会调高难度（调小目标值），反之调低。

## 交易单位

比特币的交易最小单位为聪(Satoshi)，`1 BTC = 10^8 Sats`（1亿聪）

## 参考

- Wikipedia：[比特币历史](https://zh.wikipedia.org/wiki/%E6%AF%94%E7%89%B9%E5%B9%A3%E6%AD%B7%E5%8F%B2)
- [区块链启示录：中本聪文集](https://book.douban.com/subject/30338899/)（The Book Of Satoshi: The Collected Writings of Bitcoin Creator Satoshi Nakamoto）（[读后感](https://mirror.xyz/wtfacademy.eth/Jezo3dZ_9IJ6yclzT_M21Y_njZaohfy9obiPRlemy8s)）
- [精通比特币](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/)
- 各种语言的白皮书：<https://bitcoin.org/en/bitcoin-paper> （[李笑来翻译的中英对照版](https://lixiaolai.com/#/bitcoin-whitepaper-cn-en-translation/Bitcoin-Whitepaper-EN-CN.html)）
- [以太坊白皮书](https://ethereum.org/zh/whitepaper/)
- [比特币准官网的开发者指南](https://developer.bitcoin.org/devguide/block_chain.html)
- 最新源码：<https://github.com/bitcoin>
