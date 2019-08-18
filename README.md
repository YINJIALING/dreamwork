首先我是参考的一位大佬的github,这里给出连接，感谢他
https://github.com/Snailclimb/JavaGuide
https://github.com/jwasham/coding-interview-university/blob/master/translations/README-cn.md?utm_source=wechat_session&utm_medium=social&utm_oi=679309733035511808
然后觉得有一些没有概括到，把增加的内容补充在这里
## java源码
1.string为什么被设计为final类的？
首先final表示为不可变，并不是指的是字符串值不可变，可变的是字符串的引用，当string str="jfjs"其实在常量池中开辟一块存放后边的值，str指向该值。
被设计为final出于以下3点考虑：1.要存放在常量池中，为什么要有常量池？相同的字符串存放在常量池给堆节省很多空间2.保证了多线程安全，假设像数据库密码或端口号一类的字符串可被修改，会导致很多问题3.保证了hashcode不变。在创建时string的hashcode就被缓存了，那么在计算的时候不需要重新计算，大大提升了执行效率
string.intern：若字符串存在返回常量池的引用。否则创建并返回引用。

2.反射forName和classLoader区别
classLoader只是把类加载到内存中并不初始化(static代码块不被执行)；forName不仅把类加载到jvm中，还对类进行解释，执行了static方法

3.对于null的理解

不是对象，也不是类型。仅仅用于表示该对象没有指向的引用，用途：将某个对象置为null等待jvm去回收；
用的时候需要注意：
- 只能通过==和!=进行运算
- 不能赋值给基本类型；只能赋值给引用类型

4.map key is null

hashmap:key,value都可以为null

hashtable:key,value都不可以为null

treemap:允许value是null,但不允许key 是 null

linkedhashmap:同 hashmap

concurrenthashmap:同hashtable(原因：因为hashtable和concurrenthashmap是线程安全的，假设支持，当使用get(key)返回null无法判断是真的value为null还是这个 key没作映射)

5.异常处理
所有的异常都是throwable的子类，分为error和exception。error出现标志着系统发生不可控的异常，如:stackflowerror, out of memory必须要人工介入。exception分为checked和unchecked.checked exception指的是在代码中需要显式处理的代码，如：classNotFoundException,sqlException. unchecked exception指的是运行时的异常，如:nullpointerException,indexoutboundsException

6.aqs抽象同步队列
- violate state用于描述一个共享状态，判断当前是否能同步。要点：1.setState和getState用final修饰，限制aqs的子类重写这两个方法。2.用 cas的方法setstate,也是用final修饰
- 同步队列：一个先进先出的双向队列，分别用 head和tail标记头和尾，结点的类型是node。当线程获得临界资源失败便会把这个线程构造一个node加入这个队列，同时阻塞这个线程。直到临界资源释放，唤醒head.（node:一个node表示一个线程，还保存着线程的状态，线程的引用， prev,next）
- conditionObject:实现等待通知机制。conditionObject实现了condition接口，给aqs提供条件变量的支持。与同步队列的关系：调用了await的线程会加入conditionObject等待队列中，并唤醒同步队列中的next节点。线程在某个conditionObject对象上调用了singal方法，等待队列中的firstWaiter会被加入到同步队列中，等待被唤醒。当线程调用unLock释放锁的时候，同步队列的header的下一个结点会被唤醒。
与同步队列的区别：都是维护了一个单独等待的队列，结点类型都是node（node:一个node表示一个线程，还保存着线程的状态，线程的引用， nextWaiter）头和尾firstWaiter,lastWaiter
- 独占锁与共享锁：独占锁：只有一个线程能进入临界资源，如：reentrantLock,分为公平锁和非公平锁。共享式：信号量，countdownlatch
- 模版方法设计模式：在父类中定义一些步骤，具体执行放在子类中，使得子类可以在不改变结构的情况下，重新定义步骤。aqs定义的一些模版方法：tryRealese(独占，尝试释放资源),tryAcquire(独占尝试获取资源)，tryAcquireShared,tryrRealeseShared.
- 自定义同步器的步骤：先确定是独占锁还是共享锁，定义state的含义，再定义一个类去继承aqs,重写对应的模版方法。

## 网络
1.5层网络模型
- 物理层：提供传输的物理介质
- 数据链路：为物理层提供传输
- 网络层：定向路由，ip,icmp( ping的原理，用于评估网络连接的状态)
- 传输层：提供可靠或不可靠的传输,tcp,udp
- 应用层：将传输的数据以规定的形式显示，http,dns,talent,

2.tcp为什么要3次握手？
- 保证信息对等：当client1向client2发出请求连接时，client1不知自己的发报能力和收报能力也不知对方的发报和收报能力。当clinet2收到client1的请求，client2知道自己的收报能力和对方的发报能力。接着，client2向client1发出确认消息，此时client1知道自己的发收报能力和对方的发报和收报能力，向client2发出确认，client2收到确认，知道了自己和对方的收发报能力
- 防止超时出现脏数据：当只有2次握手时，当有个从client1出发的请求连接发出但由于网络原因没有达到client2,client1超时重传，发送成功到client2,client2向client1发送ack,连接建立。当数据传输完毕，关闭连接。clinet2收到之前网络延迟没收到的client1,以为client1又要建立连接，于是向clien1发出ack,client1由于处于listening状态，会丢弃client2发来的请求。client2单方面创建请求成功，处于established状态

3.close_wait干嘛？太多次，应该怎么解决？
- 干嘛？1.用于等client2接受到client1最后的数据2.等待client1处理最后的资源
- 太多次怎么解决？1.设置服务器参数把time_wait调小2.查看程序是否编写有误，如忘记关闭流处理资源

4.从编程的角度来看，怎么实现tcp的？
通过一个套接字，创建文件描述符fd，请求的发送和接受是因为fd调用的函数不一样。而所能创建的连接大小取决于fd的个数

5.了解的攻击方式
- sql注入：在进行拼sql的时候，直接讲未转义的字符拼接成字符串。如：select * from t_user where username="",传入1,导致返回所有的数据。解决方案：1. 使用参数化的sql语句2 过滤用户输入的特殊字符
- XSS:利用前端或后端的漏洞，向正常的网页插入恶意脚本，从而可以执行任何脚本。解决方案：1. 过滤用户输入的特殊字符
- CORS 跨域攻击脚本：利用用户对浏览器的信任，在浏览网页的时候不知觉的时候访问了恶意请求，如；转账发邮件等。解决方案：1.让用户知道自己在干嘛，发送验证码2.使用token

6.https为什么安全？
在传输层和应用层加入了ssl层，通过使用非对称建立连接+对称通信方式访问。还有ca证书
流程：首先，服务端把自己的相关信息告诉ca证书机构，ca认证后将数字签名返回给服务端。当客户端访问服务器时，服务器将自己的公钥和证书返回给客户端。客户端收到后，验证证书，证书验证通过后，生成自己的共享公钥用服务器的公钥加密后发给服务器，服务器收到后，用自己的私钥解锁，将请求的响应用共享公钥加密返回给客户端。客户端收到解密信息用共享公钥解密。

## jvm
1.字符码
机器码是0和1组成，只能由机器识别的信号，与底层硬件系统耦合。由于java的一次编码到处执行，是在底层与java源代码之间加了一层中间层.class,也就是字节码，因而可以实现java的跨平台。在代码执行的时候， jvm将字节码解释执行，编译成机器码。
2.魔数
机器码的起始4个是魔数(coffee baby),作用：标志这个文件是一个java文件。如果没有说明它已经被损坏或不是java文件。

3.如何将源码转变为字节码？
java源文件-词法分析-语法分析-语义分析-生成字节流-字节流

4.字节码执行的3种模式
- 解释执行
- jit编译执行：动态编译
- jit编译与解释执行混合(主流jvm默认)

5.源码到机器码执行过程
源码-字节码.class-classloader(将.class加载到内存中，5个阶段)- 执行(解释，jit)

6.什么情况下可以自定义类加载器
- 隔离加载类：在某些框架内进行中间件与应用的模块隔离 ， 把类加载到不 同的环境。比如 ， 阿里内某容器框架通过自定义类加载器确保应用中依赖的 jar包不 会影响到中间件运行时使用的 jar 包。
- 修改类加载方式： 类的加载模型并非强制 ， 除 Bootstrap 外 ， 其他的加载并 非 定要引入 ， 或者根据实际情况在某个时间点进行按需进行动态加载。
- 扩展加载源。比如从数据库、网络 ，甚至是电视机机顶盒进行加载。
- 防止源码泄漏， java代码容易被编译和篡改，可以进行编译加密。 

- 实现自定义加载器的步骤：定义一个classLoader继承classloader,重写findClass(),调用defineClass()（没打破双亲委派，无法被父类加载器加载的类最终会通过这个方法被加载）

如果想打破双亲委派模型则需要重写loadClass()方法（当然其中的坑也不会少）。典型的打破双亲委派模型的框架和中间件有tomcat与osgi。

7.内存布局
- heap
- stack：局部变量表，操作栈，动态连接(引用)，方法的返回地址
- native stack
- pc  
- metaspace元数据区：1.8之前叫做永久代
除pc外都会发生oom(out of momery) ，堆和元数据都是线程共享的

8.实例化发生了什么？(new Object())
- 确认去元数据区确认类的元信息是否存在，如果不存在，则通过双亲委派机制去加载，加载不到就抛出classNotfoundException,如果找到就动态生成class对象，存放到元数据区
- 去java堆中开辟空间，如果实例成员变量是引用变 量，仅分配引用变量空间即可，即 4 个字节大小，接着在堆中划分一块内存 给新对象。在分配内存空间时，需要进行同步操作，比如果用 CAS ( Compare And Swap )失败重试、区域加锁等方式保证分配操作的原子性。
- 给它的属性赋初值
- 设置新对象的晗希码、 GC信息、锁信息、对象所属的类元信息等。这个过程的具体设置方式取决于 jvm实现。
- 执行初始化方法，如给初始化变量赋值，执行实例化变量，执行构造方法，并把堆内的首地址赋值给引用变量

9.垃圾收集器
- serial:串形收集，新声代使用copy, 老年代标记整理.使用时，要stop the world,停顿时间比较长
- g1
- cms
- zgc

10.jvm调优
- 工具：visualVm
— 指令：jstat:监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。jmap(JVM Memory Map)命令用于生成heap dump文件，如果不使用这个命令，还阔以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件。jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。

## 数据库
1.发生慢查询语句怎么办？
- 增加有效索引
- 修改应用层的业务逻辑
- 使用其他数据库，如非关系型数据库
- 不使用join语句比较多的sql,
- 拆分临时表

2.在SQL标准中，前三种隔离级别分别解决了幻象读、不可重复读和脏读的问题。那么，为什么MySQL使用可重复读作为默认隔离级别呢？
首先，主从复制，是基于binlog复制的。binlog有几种
statement:记录的是修改SQL语句
row：记录的是每行实际数据的变更
mixed：statement和row模式的混合
那Mysql在5.0这个版本以前，binlog只支持STATEMENT这种格式！而这种格式在读已提交(Read Commited)这个隔离级别下主从复制是有bug的，因此Mysql将可重复读(Repeatable Read)作为默认的隔离级别。当在master上执行的顺序为先删后插！而此时binlog为STATEMENT格式，它记录的顺序为先插后删！从(slave)同步的是binglog，因此从机执行的顺序和主机不一致！就会出现主从不一致！
如何解决？
解决方案有两种！
(1)隔离级别设为可重复读(Repeatable Read),在该隔离级别下引入间隙锁。当Session 1执行delete语句时，会锁住间隙。那么，Ssession 2执行插入语句就会阻塞住！
(2)将binglog的格式修改为row格式，此时是基于行的复制，自然就不会出现sql执行顺序不一样的问题！奈何这个格式在mysql5.1版本开始才引入。
3.数据库范式

关系：要满足第3范式，必须先满足第2范式，要满足第2范式，先满足第1范式
- 第一范式：无重复列，指实体中任一属性不能重复分，像地址：江苏省苏州市人民路，就不符合第一范式，拆分成省：江苏市：苏州，街道：人民路就符合第一范式。
- 第二范式：非主属性完全依赖主属性。如：有一张选课记录表，字段有：姓名，年龄，课程名，学分，成绩，就不符合第二范式，因为对于同一个学生就有m门课，姓名和年龄重复了m-1次，对于同一门课有n个学生选，课程名和学分就重复了n-1次。符合第一二范式的做法是，拆分成3张表，学生表：姓名，年龄，学生id;课程表：课程id,课程名，学分；选课表：学生ID，课程 ID，成绩。这样的好处是：不用多次更新（假设要修改学分，则需要更新n-1条，更新异常；数据冗余；插入异常，如果想要存一门课程只有课程名和学分，无法插入）
- 第3范式：主键不依赖其他非主属性。在一张部门表中，有部门id,部门名，简介等，员工表中有员工名，员工号，部门ID。员工表中就不能有其他部门的信息，否则会造成数据冗余

4.慢查询explain显示什么
相关问题：如何查看一个数据库是否用了索引
思路：explain的关键字
id(表的执行顺序，序号越大越优先执行，序号相同从上往下执行),select_type (查询类型，是否有子查询，简单查询simple),table,partitions,type(索引类型： const:返回一条数据，主键索引或唯一索引；eq_ref:唯一索引，返回一条数据;ref:非唯一性索引，返回多记录；range:范围查询，where后面sh是><between and,(最好不要提到in)；),possible_keys(可能用到的索引，不准确),key(实际用到的索引),key_len(索引的长度，用于判断索引是否全部被用到),ref,rows(实际通过索引查到的数据个数),filtered,extra
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
## 设计模式
1.简单工厂模式：将对象的创建推迟到工厂中，如：有个shape接口，有rectangle,circle实现了这个接口，有个shapefactory类，当需要创建实例对象的调用shapefactory类的create方法，传入参数用于表示创建什么实例对象，根据这个参数创建对象。好处：将创建对象的放到工厂类中创建，解耦。
工厂方法模式：同样也是将对象的创建推迟到工厂中，只是有个factory接口， 要创建对应对象的工厂类并实现factory接口，如创建rectangle用rectanglefactory类，创建circle用circleFactory类。想要新增一个图形star,要先创建star类实现shape接口，然后创建starfactory实现factory接口。
抽象工厂模式：增加了产品族的概念，如：要实现一个red的circle的实例,先创建shape接口和color接口，再创建circle实现shape接口以及red实现color接口，接着创建factory接口，redcirclefactory实现factory接口，他有2个方法分别是create和colored,用redcircle初始化完成。好处：提供了一个更加复杂的产品的构造。

## Linux文件系统
1.虚拟内存 调度方式 存储地址变化方式(还没看懂)
虚拟内存:让应用程序觉得在使用的是一整块内存，实际上是很多块物理碎片。window,linux都用到了
调度方式：
- 页式：将实存和虚拟内存都分为页号和页内地址，每个程序设置一张页表，通过页表来访问实际地址。缺点：页的大小固定，无法按照程序的大小自适应。优点：便于管理
— 段式：每个程序设置段表，每个段包含的字段：是否有效(是否加载到内存中)，段起始位置，端偏移位置。可以根据程序的大小调整。
- 段页式：段+页结合，兼容2者的优点。
2.文件系统

3. 进程和线程的区别
举word的例子，然后总结
- 进程有自己独立的内存，线程共享同一个进程的内存
- 一个程序至少一个进程，一个进程至少一个线程
- 进程通信比较困难，线程因为共享内存通信比较简单
- 进程是cpu资源分配的最小单位，线程是cpu资源调度的最小单位


还没弄清楚的问题
1.一条sql的执行过程

6.liunx指令
7.事务是怎么用动态代理做的
8.redis的分布式锁实现是基于什么的？单线程，多个线程竞争一把锁，谁竞争到谁先就先拥有锁
9.innodb是否都是聚集索引？是，如果没主键，会自动生成一个隐藏的列，还是生成聚集索引




