首先我是参考的一位大佬的github,这里给出连接，感谢他
https://github.com/Snailclimb/JavaGuide
然后觉得有一些没有概括到，把增加的内容补充在这里

## Redis
1 如何保证高可用？
- 集群：拥有原生的集群机制(memcache没有)
- 主从复制：
什么是主从复制？当master写入，也写入slave,并且不会阻塞master,master可以继续处理client请求，保持slave数据与master数据实时同步.保证高可用，避免单点故障
实现机制：1）完全复制： slave向master发出同步的命令，master后台生成一个rdb文件，并把该期间的写入操作存在缓冲区。生成完成，slave利用rdb进行同步，保持slave状态与master一致，并把缓冲区的数据写入到slave。缺点当slave重启，master不管slave有多少数据与他一致，都会重新进行同步，性能差2）部分复制：slave向master发出部分同步的命令，并携带master的机器编码以及offset， master收到后，验证机器编码是否与本机一致，不一致就完全复制，否则就按 offset进行复制。
- 持久化
- 哨兵：为了解决主节点挂了，由新的主节点修改配置。过程：由sentiel node向各个节点发送ping命令，如果master在规定时间内没有收到反馈。那么master就被标记为主观下线，如果master被标为主观下线，如果超过半数的sentiel node向 master发请求没收到回应，那么master就被标记为客观下线。此时，sentiel node和其他sentiel node协商master的状态，如果master处于sdown状态就投票自动选出新的节点，并将其他的slave指向新的结点，当没有足够数量的sentiel node同意 master下线，主服务器的客观下线状态就会被移除，当主服务器向sentiel node返回响应，主观下线状态就会被移除。
2.实现分布式锁

## RabbitMQ
1.镜像队列
2.如何确保消息传递收到
publisher确认机制

## Linux文件系统



