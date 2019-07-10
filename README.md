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
- redlock: 分布式环境下，假设有N个Redis master。这些节点完全互相独立，不存在主从复制或者其他集群协调机制。我们确保将在N个实例上使用与在Redis单实例下相同方法获取和释放锁。现在我们假设有5个Redis master节点，同时我们需要在5台服务器上面运行这些Redis实例，这样保证他们不会同时都宕掉。为了取到锁，客户端应该执行以下操作:

获取当前Unix时间，以毫秒为单位。

依次尝试从5个实例，使用相同的key和具有唯一性的value（例如UUID）获取锁。当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试去另外一个Redis实例请求获取锁。

客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。当且仅当从大多数（N/2+1，这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。

如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。

如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）
- setkey value px milliseconds nx:set命令要用 setkey value px milliseconds nx；value要具有唯一性；释放锁时要验证value值，不能误解锁；事实上这类琐最大的缺点就是它加锁时只作用在一个Redis节点上，即使Redis通过sentinel保证高可用，如果这个master节点由于某些原因发生了主从切换，那么就会出现锁丢失的情况：在Redis的master节点上拿到了锁；但是这个加锁的key还没有同步到slave节点；master故障，发生故障转移，slave节点升级为master节点；导致锁丢失。


## RabbitMQ
1.镜像队列
如果RabbitMQ集群只有一个broker节点，那么该节点的失效将导致整个服务临时性的不可用，并且可能会导致message的丢失（尤其是在非持久化message存储于非持久化queue中的时候）。可以将所有message都设置为持久化，并且使用持久化的queue，但是这样仍然无法避免由于缓存导致的问题：因为message在发送之后和被写入磁盘并执行fsync之间存在一个虽然短暂但是会产生问题的时间窗。通过publisher的confirm机制能够确保客户端知道哪些message已经存入磁盘，尽管如此，一般不希望遇到因单点故障导致服务不可用。镜像队列机制就是将队列在三个节点之间设置主从关系，消息会在三个节点之间进行自动同步，且如果其中一个节点不可用，并不会导致消息丢失或服务不可用的情况，提升MQ集群的整体高可用性。
2.如何确保消息传递收到
publisher确认机制，2种方式
- 通过AMQP事务机制实现(3个方法)：txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务，在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定到达了broker了，如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。
- 通过将channel设置成confirm模式来实现：生产者将channel设置成confirm模式，一旦channel进入confirm模式，所有在该channel上面发布的消息都会被指派一个唯一的ID(Correlation Id从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）,这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。异步 的方式

## Linux文件系统
- 



