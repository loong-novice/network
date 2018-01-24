＃网络
BGP属性 研究

本人 网络小白一名 初学者 有理解不对的 希望大家批评指导

![topo](https://github.com/loong-novice/network/blob/readme-edits/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_126244db-33cc-4524-8aab-b192631a6bc7.png)

研究 ： BGP -med属性   local-pre属性 

a   各个IBGP EBGP 用lookback0 口建立邻居关系（不用直连口）

b   在BGP中引入ospf 

c   调节med值 使在R1上看到R4的路由 优选R3这条路径


IP地址规划 例如 r1与r2 连接  r1的接口IP为 12.1.1.1 24 r2 为12.1.1.2 24  r1 loopback0 1.1.1.1 32 r2  loopback0 2.2.2.2 32


基本配置 在后续上传的文件里 这里大体说一下几个重点
首先用环回口做更新源（做peer）因此配置的时候需要 加上 peer XXXX connect-interface loopback0

![bgp-connect](https://github.com/loong-novice/network/blob/readme-edits/bgp%20connect.png)
这里 我建立一个group 100 是 internal（表示IBGP） 在r2上还多存在一个group 200 作为ex（表示 EBGP）
在r1上看 BGP peer（r2 r3） 建立成功  

![peer-est](https://github.com/loong-novice/network/blob/readme-edits/bgp-peer.png)

 在r2 上看peer 发现 IBGP 邻居建立成功，但是EBGP出现Idle  

![peer-idle](https://github.com/loong-novice/network/blob/readme-edits/peer-idle.png)
 
 原因在于 在用loopback建立EBGP时候 需要修改ebgp-max-hop 当BGP使用loopback口建立EBGP对等体时候，必须指定本参数>=2
 否则对等体无法建立。当 hop=1时候 表示建立EBGP是直连对等体。（loopback 口可以当成 一个终端来看 因此hop>=2）
 修改后对等体建立成功
 
 ![R4-peer](https://github.com/loong-novice/network/blob/master/r4-peer-est.png)
 
 由于所有路由都在ospf中宣告，包括直连网段加上loopback 网段 然后在BGP中引入直连与ospf
 由于med 是用于EBGP对等体判断流量进入其他AS时候的最优路由 也就是在r4 上宣告的 4.4.4.4 在r1 可以看到两条等价路由 然后修改med值
 正常情况下会优选r2 因为最后比较到Route-id小的。我们先 在r1  dis bgp routing-table  
 ![r1-bgp-routting](https://github.com/loong-novice/network/blob/master/r4-peer-est.png)
 
最优路由竟然是 0.0.0.0 怎么回事呢？
这说明是本地始发路由，但是在拓扑中该路由的目的网段4.4.4.4 并没有直接连接在BGP speaker上呀？
其实这条路由优选了ospf 因为ospf的外部路由优先级高于ibgp ebgp ospf--10  IBGP---255 Ebgp--255
在r1上看路由表就清楚了，dis ip routing-table

![r1-ip routing](https://github.com/loong-novice/network/blob/readme-edits/r1-ip%20routing.png)

其实解决办法有两个 
1 其实是业务网段与互联段没有规划好 例如 4.4.4.4 应该是业务网段，应该在r4中 BGP中 network 出来 而不是在ospf中network 在import
2 此办法在实际中不推荐使用，但是在实验环境下取很有意思，就是修改ospf的外部优先级，把ospf优先级以及IBGP EBGP 优先级低 让其通过BGP import 获取到4.4.4.4 同理在r4上看r1的1.1.1.1 路由一个原理

我们先来看第二个解决办法 
分别在 R1 R2 R3 R4 上调节 ospf 与BGP 优先级ospf   preference 250 preference ase 250   bgp preference 120 120 120 

![图2](https://github.com/loong-novice/network/blob/readme-edits/%E5%9B%BE%E4%BA%8C.png) 

这里默认选择了r2 上的路径
这时我们修改r2的med 数值  先创建一个 route-policy 然后在BGP中 使用策略 
route-policy in-bgp permit node 10
apply cost 66 
在BGP中 peer 4.4.4.4 route-policy in-bgp export 
这时候在r4 上看1.1.1.1 的路由 就应该优选 3.3.3.3  

![图3](https://github.com/loong-novice/network/blob/readme-edits/%E5%9B%BE%E4%B8%89.png) 


以上配置为配置一文件


第一个解决办法  就是用互联接口建立peer 这时候有人问为什么不继续用环回口，首先在ospf中宣告互联网段，在BGP中宣告环回口地址，这样建立不起来peer
因为在ospf中无法有到达环回口路由，可以写一条静态路由 这样繁琐不说 如果是单出口，比如r1就上联或者下联一台r2，可以写一个默认路由，把无法匹配的路由全撇到出口，在通过直连获取到r2环回口
这里我们用互联接口做peer 在BGP中宣告环回口（可以理解为业务网段） ospf 宣告互联网段 具体配置在 配置二 （此种方法 难度不大 也都是基本配置）
在 r2 上 还是bgp设置 export 策略  也就是r2发给r4的路由 med 值都为50 
peer 24.1.1.4 route-policy in-bgp export 

在r4上看r1的 1.1.1.1 路由 应该首选r3这条路径 
![r4](https://github.com/loong-novice/network/blob/readme-edits/r4.png)


如果 在r2上写 peer 24.1.1.4 route-policy in-bgp import 表示收r4发给r2的路由 在经过r2后修改med 这时候就是在r1上看r4（4.4.4.4）的路由 

未修改med

![r1-bef](https://github.com/loong-novice/network/blob/readme-edits/r2--med-before.png)

修改后med

![r1-after](https://github.com/loong-novice/network/blob/readme-edits/r2-med-import.png)


也可以修改local-pre 来进行选路 值越大越优选  在r2上在设置


![local-be](https://github.com/loong-novice/network/blob/readme-edits/bgp%20local%20%E5%89%8D.png)

![after](https://github.com/loong-novice/network/blob/readme-edits/BGP%20local%20%E5%90%8E.png)
 


如果在r3上也修改med 就是 r3 med>r2 med  但是r3的locprf < r2 locprf  这时候看选路优选那个?



![gai](https://github.com/loong-novice/network/blob/readme-edits/r3%E4%BF%AE%E6%94%B9%20%E5%88%A4%E6%96%ADmed%20local.png)
 

实验表明 local 优先级 大于 med
 
 













