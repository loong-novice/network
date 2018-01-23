＃网络
![topo](https://github.com/loong-novice/network/blob/readme-edits/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_126244db-33cc-4524-8aab-b192631a6bc7.png)

研究 ： BGP -med属性   local-pre属性 

a   各个IBGP EBGP 用lookback0 口建立邻居关系（不用直连口）

b   在BGP中引入ospf 

c   调节med值 使在R1上看到R4的路由 优选R3这条路径


IP地址规划 例如 r1与r2 连接  r1的接口IP为 12.1.1.1 24 r2 为12.1.1.2 24  r1 loopback0 1.1.1.1 32 r2  loopback0 2.2.2.2 32


基本配置 在后续上传的文件里 这里大体说一下几个重点
首先用环回口做更新源（做peer）因此配置的时候需要 加上 
peer XXXX connect-interface loopback0



