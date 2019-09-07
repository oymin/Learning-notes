# RabbitMQ 集群恢复与故障转移

RabbitMQ 镜像队列集群的恢复的解决方案和应用场景：

前提：比如两个节点 A 和 B 组成一个镜像队列

## 场景1：A 先停, B 后停

方案一：该场景下 B 是 Master, 只要先启动 B, 再启动 A 即可。或者先启动 A, 再30秒之内启动 B 即可恢复镜像队列。

## 场景2：A, B 同时停机

方案二：该场景可能是由于机房掉电等原因造成的，只需在30秒之内连续启动 A 和 B 即可恢复镜像。

## 场景3：A 先停, B 后停, 且 A 无法恢复

方案三：该场景是1场景的加强版，因为 B 是 Master, 所以等 B 起来以后，在 B 节点上调用控制台命令 `rabbitmqctl forget_cluster_node A` 接触与 A 的 Cluster 关系，再将新的 Slave 节点加入 B 即可重新恢复镜像队列。

## 场景4：A 先停, B 后停, 且 B 无法恢复

方案四：该场景是场景3的加强版，比较难处理，原因是因为 Master 节点无法恢复，早在 3.1x 时代之前没有什么好的解决方案，但是现在已经有解决方案了，在 3.4.2 以后的版本。因为 B 是主节点，所有直接启动 A 是不行的，当 A 无法启动的时候，也就没办法在 A 节点上调用之前的 `rabbitmqctl forget_cluster_node B` 命令了。新版本中 `forget_cluster_node` 支持`--offline` 参数。

这就意味着允许 `rabbitmqctl` 在理想节点上执行该命令，迫使 RabbitMQ 在未启动 Slave 节点中选择一个节点作为 Master。当在 A 节点执行 `rabbitmqctl forget_cluster_node --offline B` 时，RabbitMQ 会 mock 一个节点代表 A，执行 `forget_cluster_node` 命令将 B 剔除 cluster，然后 A 就可以正常的启动了，最后将新的 Slave 节点加入 A 即可恢复镜像队列。

## 场景5：A 先停、B 后停，且 A、B 均无法恢复，但是能得到 A 或 B 的磁盘文件

方案五：这种场景更加难处理，只能通过恢复数据的方式去尝试恢复，将 A 与 B 的数据文件模式在 `$RABBIT_HOME/var/lib/` 目录中，把它拷贝到新的节点对应的 mulxia，再将新的节点 hostname 改成 A 或 B 的 hostname，如果是 A 节点（Slave）的磁盘文件，则按照场景4处理即可，如果是 B 节点（Master）的磁盘文件，则按照场景3处理即可，最后新的 Slave 加入新节点后完成恢复。

## 场景6：A 先停、B 后停，且 A、B 均无法恢复，且得不到 A 和 B 的磁盘文件

准备甩锅吧！！！

