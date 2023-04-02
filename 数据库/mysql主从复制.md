1. 配置主节点 / 从节点信息
2. 从节点，开启 I/O thread 监听主节点的Binlog日志
3. 主节点Binlog日志发生变化，I/O thread 获取变化数据，写入 中继日志（Relay log）
4. 再由SQL thread 根据Relay log, 将数据写入 从节点数据库
5. 告知主节点，完成数据复制