＃网络
BGP属性 研究

本人 网络小白一名 初学者 有理解不对的 希望大家批评指导


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
 
 








