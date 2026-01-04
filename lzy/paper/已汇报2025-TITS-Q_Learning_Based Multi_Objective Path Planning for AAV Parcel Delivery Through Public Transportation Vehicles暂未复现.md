参考文献18investigates how swarm intelligence, combined with smart city infrastructure, can enhance drone logistics while addressing challenges like com munication, battery limitations, and urban safety. I

参考文献19presents a hybrid genetic algorithm combined with an enhanced Dijkstra to minimize energy consumption and ensure reliable communication during multi-AAV delivery operations. 

无人机受电池容量限制，可以适应公交车时刻表和连接质量的变化。

最小化提前到达时间、能量消耗和累积干扰

累计干扰包括：来自其他用户 UE 的累计干扰，来自其他基站的累计干扰

因此累计干扰越多，SINR越小，导致连接差，容易中断

V1：代表所有节点

E1：代表边

V2：代表公交车停靠点

E2：代表连接站点的边

li：代表所有公交车线路

tj：表示某条线路上的第j个班次行程

Tli:为线路li的所有班次的集合

 θlitjvv‘：两个节点间的通行时间

Tc:任务完成时间，即无人机抵达客户c所需的时间

Pc：为从w到c的所有可能路径的集合。

ef：无人机在飞行阶段消耗的能量

eh：无人机在悬停阶段消耗的能量

Wt: 表示在当前时间、当前节点 v上的悬停时间。





输入：

S、A

State:状态 s=(v,t)

v：表无人机当前所在节点

t：当前时间



Action：动作

a=<li,tj>

li=l0,无人机自己飞

li!=l0,无人机乘坐公交车

无，悬停等待



无人机有电池容量限制，不考虑充电，一次仅配送一个货物，货物重量相同，









模拟：

公交网络图如图3

5个站点和5条公共交通线路

模拟区域3km*2km

用户设备数(ue)：50-200

5-10个用户组内按10-20个ue大小分组

模拟基站（BSs）:3-10

用户速度:0.5-5m/s



1：在s1









