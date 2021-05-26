# ClusterChatSever



# How to use it ? 
1. 安装mysql， 更改root密码为root, 设置监听地址为所有，而不是本地， 并且mysql -u root  -p 的命令前面不要加sudo就能进去。 不然代码接口访问不了mysql。 此外还要安装 sudo apt-get install libmysqlclient-dev ， 这是开发的依赖库。 
2. git clone 代码， 将代码拷贝到本地。 
3. 安装配置nignx
- + ./configure --with-stream ， 缺什么安装什么 sudo apt-get install libpcre3 libpcre3-dev   sudo apt-get install openssl libssl-dev

- +  make && make install   编译完成后，默认安装在了/usr/local/nginx目录。
- +  修改nginx配置文件内容为ClusterChatSever/test/conf/nginx.conf中的内容。 
- + 这里除了自己把字母拼错， 出现了两个问题，一个是没有shared 库， 这里面我就参照【3】做了，然后又出现了 [error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)问题， 参照【4】解决。最后进入安装的nginx/bin 中 nginx -s reload成功。 
4.  配置redis  这里就不介绍了， 和nginx差不多， 记得在cmake中添加依赖库(我已经添加了)。 
5. 运行调试， 发现之前ginx设置的断开链接事件太短了。 需要注意一下（已经修改）。 最后得出来运行的效果。
 

# 介绍



## 1 背景
 秋招在急，简历中没有一个合适的项目供面试官去展开提问和自己去练习。  而且在学习c++的过程中最大的问题就是没有合适的项目去融汇贯通， 不知道做什么才好，像java可以做很多有意思的后台项目， 但是c++貌似如果只是懂语法去做这些后台项目很难， 不知道从什么点出发去做。我想通过这个项目去掌握C++如何开发后台项目，不涉及分布式。  通过这个项目明白后台的开发流程， 让自己的c++能够做到实际开发的水平， 敢写c++。
 
 ## 2 任务
实现一个集群聊天服务器， 能够高并发的接收客服端的请求。 通过这个项目掌握
1. 掌握服务器的网络I/O模块，业务模块，数据模块分层的设计思想， ORM
2. 掌握C++ muduo网络库的编程以及实现原理
3. 掌握Json的编程应用
4. 掌握nginx配置部署tcp负载均衡器的应用以及原理
5. 掌握服务器中间件的应用场景和基于发布-订阅的redis编程实践以及应用原理
6. 掌握CMake构建自动化编译环境
7. 掌握Github管理项目
8. 熟悉现代c++， 尤其是绑定器和function等使用， 掌握面向对象设计的思想。 


 ## 3 行动

 ### 1. 环境配置
 1. vscode + 远程linux

 ### 2. 框架
  整体的框架如下， 多个服务器通过redis构建互通信息， 然后将多个服务器绑定到niginx上，与客户端交互。 服务器采用muduo库作为网络库底层， 以muduo库提供的回调作为业务层， 以自己封装的数据库类作为数据层的MVC结构进行开发。
最终实现了一个高并发的集群服务器。 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210522133518257.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXBlbmcxOTk3MDExOQ==,size_16,color_FFFFFF,t_70)


 ### 3. 技术栈
 C++, linux , cmake , mysql , redis , nginx , muduo , gdb , git, json



 ### 4. 具体步骤

 1. 学习json和json开源库
 - +   数据交换语言是独立于编程语言的， 不同编程语言通信时通过json等交换语言进行。 json是一个比较简单的字节流交换语言。 通过key-value存储数据。 我们使用一个开源库 ， 他可以支持很多语言的json序列化和反序列化（也就是转换成数据类型或数据类型变成json）。用的时候也比较方便， 直接倒入一个头文件就行， 具体序列化实现都在这个文件文件中做了。 


2. muduo 网络库使用和学习
-  + 编译安装参照【1】， 这里面我已经装好了。 
-  +  muduo和众多网络库都是epoll+ 线程池的高性能服务器设计。 可以让我们直接关注到连接断开读写这几个事件的callback业务层， 请求来了底层自动调用callback。 
-  + 学会使用muduo开发一个基本的高并发服务器。 

3. cmake的使用与学习
- + 在vscode上配置cmake 辅助工具(界面操作+代码提示与检查)， 便于操作linux系统中的cmake. 
- + 基本的编译程序所做的命令例如：g++ -o server -g muduo_server.cpp xx.cpp -l/usr/include -L/usr/ib -Imuduo_net -lmuduo_base -lpthread  都可以在cmake中找到对应的配置选项， 去自动生成makefile。 
- +  cmakelists.txt 真的很强， 通过add_subdirectory(src) 去找子cmakelists.txt去编译。 


4. mysql 学习
- + mysql 安装与账号、权限配置
- + 创建业务需要用到的数据库和表。
```sql
CREATE TABLE IF NOT EXISTS user (id INT  AUTO_INCREMENT PRIMARY KEY  , name VARCHAR(50) NOT NULL UNIQUE ,  password VARCHAR(50) NOT NULL  , state   ENUM('online', 'offline')  DEFAULT 'offline'   )ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE IF NOT EXISTS friend (userid INT NOT NULL , friendid  INT NOT NULL  ,PRIMARY KEY (userid,friendid) )ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE IF NOT EXISTS allgroup (id INT  AUTO_INCREMENT PRIMARY KEY  , groupname VARCHAR(50) NOT NULL UNIQUE ,  groupdesc VARCHAR(200) DEFAULT ''  )ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE IF NOT EXISTS groupuser (groupid INT  NOT NULL , userid int  NOT NULL , grouprole   ENUM('creator', 'normal')  DEFAULT 'normal'  ,PRIMARY KEY (groupid,userid) )ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE IF NOT EXISTS offlinemessage (userid INT NOT NULL , message  VARCHAR(500) NOT NULL  )ENGINE=InnoDB DEFAULT CHARSET=utf8;
```


4. 创建源代码目录
整个项目在src，include中分为后台和客户端。 各种目录的编译通过各级的cmakelist.txt去控制。 

5.  开发网络模块，搭建业务模块, 定义消息类型
- + 我们使用muduo库开发网络层，在网络层的回调中， 设置了单例的业务类， 根据自身消息的id在业务类的map回调函数表中找对应的处理函数， 至此网络模块不再变动添加，完全解耦。 
- + 消息类型如下： {"msgid" : 4 , "name" : "zhang san", "password" : "123456"}
- + 

6. 封装mysql数据库类
- +  像orm框架一样， 用类封装数据库操作， 这样当数据库的代码有所变动，例如表的变动， 不会变动业务代码。
- + 使用时候我们先将一个表定义成一个表类， 定义一个数据库连接类， 定义一个数据库封装类， 这个类包传入表类和包含数据库连接类， 在封装类中将表类中数据传递到数据库连接类（数据库连接类依赖mysql的客户端包）。
- + 简单的传入一两个数据的话，不用再添加一个数据类。 直接定义一个数据库操作类。

7. 编写业务模块类
- + 编写了数据库类之后，我们开始关注业务层。 注意数据库层一般是辅助业务层的， 所以不能在一开始就写完数据层，而是业务层缺什么补什么。 
- + 首先来看登录业务， 我们根据输入的信息 要去数据操作类中设置一个函数查找对应id的数据，判断是否账号密码匹配。 当匹配的话进入业务逻辑，例如是否已经登录等。 
- + 注册业务比较简单了， 将json数据放入user对象之后， 传入到数据库操作类定义的insert函数中对接数据库sql。
- +  添加客户端异常退出， 这个不是回调业务， 而是当服务器退出时候需要处理业务类的数据，清空在线状态。此外当我们调试阶段会主动关闭服务器， 导致里面的业务数据来不及更新。 因此添加ctrl +c 接收函数， 处理退出时候的业务重置。这个放到main函数里面。 但是好像不怎么解耦， 不过业务重置所在的类实单例业务类， 也没事。
- +  点对点聊天业务： 定义好发送信息的json格式之后，一个用户往另外一个用户发送的数据服务器收到之后会判断对方用户是否在线， 在线的话直接往对应的端口发送， 不在线的话就保存数据到离线数据库中。 
- + 离线业务比较简单，而且不是单独的业务，而是一个放在其他业务的子服务（例如当登录的时候先去调用这个数据库封装类看有无消息）。 而且其数据表比较简单， 所以我们不用设计数据对象， 直接封装数据库操作类， 传入的参数为整数和string即可。 
- + 加好友的业务 ： 当对应添加id的数据来的话， 对friend表进行insert， 此外还要将好友数据类的多表联合查询功能提供给登录业务， 一开始的时候就要显示当前登录账号的好友。 
<<<<<<< HEAD
- + 创建群的业务 ： group 是需要定义一个对象来存储数据的， group有两个表， 有一个数据库对象负责创建群的和管理这个群用户之间的关系。 
=======

8. 编写客户端
- +  完成服务器之后，我们使用简单的多线程开发一个termail客户端。 客户端通过主线程管理聊天过程， 子线程阻塞接收服务器的发送信息。 此外通过functional 绑定自身的函数， 将switch选择调用函数的耦合度较低代码做成闭合的代码。
- +   一些交互的代码这里就不再补充了。 如果出错的话或者不出错但是逻辑输出不对，记得调试一下。 

9. 加入nginx负载均衡器
- + 我们主要使用nginx的负载均衡器， 注意这里使用的是tcp负载均衡模块。 本来一台是1-2w并发， 但是nginx能够轻松5-6w， 因此多加几个应用服务器后可以达到5-6w， 此外还可以nginx可以集群， 在前面加硬件的LVS负载均衡。
- + 负载算法有轮询， 一致性哈希（在短链接中经常用，http服务器中经常问），负载权重等。 
- + 此外聊天服务是长连接服务， 服务器发送给客户端还需要进行负载均衡器。 也有直接发送client，不经过nginx的服务器。 
- + nginx还有心跳检测功能，保证服务器一直正常工作。 还有支持动态加载配置文件， 不用重启就可以更换添加应用服务器。 
- +  应用服务器启动后， nginx会自动管理他们。 我们访问nginx时候会转发给响应的服务器。 


10. redis 解决跨服务器之间的数据交互
- + 当client1和client2登录在不同的服务器上， client1的userconnmap里面找不到client2的信息， 因为虽然是在线的， 但是他登录在其他的服务器上。 这个问题怎么解决呢？ 我们先在userconnmap找， 如果找不到再去数据库找这个用户是不是online, online的话代表在其他服务器上， 那么这就还有一个问题， 如果把数据发过去呢？我虽然知道他在其他服务器上，但是不知道具体在哪个。那么得两两相互连接，每一台服务器要和其他服务器互相连接。 这会代码的耦合性非常高。因此我们使用服务器中间件中的消息队列中间件。 像比较出名的kafka（还支持分布式部署的）, rabbitmq都是， 但是我们这个项目比较简单，就只用个基于发布订阅的redis就行。（注意这里我们没有使用redis的key-value这些内容，而是用了redis的一个发布订阅的小功能。 ） 

- + 基于发布订阅的redis ： 每一个用户在chat服务器上登录时 ，都会去redis上订阅自己的id相关的消息,当别人发送消息时候，本地找不到，数据库显示在线状态， 就肯定在redis中， 就执行publish消息给redis。 redis将消息notify给订阅这个主题的人。 
- + redis首先是一个强大的缓存服务器，比memcache强大很多，不仅仅支持多种数据结构（不像memcache
只能存储字符串）如字符串、list列表、set集合、map映射表等结构，还可以支持数据的持久化存储
（memcache只支持内存存储），经常被应用到高并发的服务器环境设计之中。
- + 安装redis服务器，客户端登录后测试，并且安装客户端源码，里面包含了开发库和头文件。  
- +  如何将redis的代码加入到之前的项目中呢？ 在登录的时候连接redis，设置上报时候的回调函数（bind+fc） des订阅主题。 单点或者群聊天的时候如果online本地找不到的话记得加publish。 上报设置的回调是别人发送消息时候redis服务器notice过来时候会自动调用的。 回调里面是在这个服务器的con连接中找到对方要发的con， 进行推送。 
 


## 4 结果
完成服务器的搭建。 高并发高可用。 


## 5 问题记录

### 编程问题

1. define 内容别的地方引用时候要在头文件里面define才能用到。 
2. g++  muduo_server.cpp  -o server -lmuduo_net  -lmuduo_base -lpthread  这个连接库是有依赖关系的， 最基础的在最前面。 
3. sudo  netstat  -tanp 查看端口对应的进程。 
4. 回调之前一直不理解， c++回调基本全部用bind+ functional 去代替了， 因为普通函数回调限制较多不能携带使用类变量。 回调贯穿了整个程序oop的解耦操作中， 当一个函数什么时候发生和发生时候怎么做不再一起，就要事先设置回调。 供其他程序调用发生时候启动。（其实可以写到其他程序那部分， 但是不能解耦。）这个项目中我们将业务封装到了业务类中， 当网络这块回调被epoller-wait启动后根据数据的id拿到业务类的对应方法进行调用(通过bind+ fc+ map)。 业务再怎么改， 这边代码都不动了。 真正解耦。 
5. 头文件负责定义类还有添加头文件（定义时候需要用到的） ， 注意只需要编译一次。 对应的.cpp 负责具体实现， 当我们将其编译成库的时候， 就只需要头文件了， 具体源码实现都在.so中， 不用库的话， 也会自动去找.cpp的实现。当然我们有时候不需要这个cpp文件，类直接在hpp里面定义并声明完成。  
6. 当形参变量和类成员变量的名字一样时候，一定要加this区分。 
7. 如何设计开源文件的目录， bin放可执行文件  ， lib是生成的库文件 ， include是头文件， src 是源码， build 项目编译时候产生的临时文件， test放的代码， cmakelist.txt 设计编译文件的规则,  autobuild.sh 自动编译， readme.md 。 
8. 公网链接的时候， 服务器绑定自己内网网卡的地址， 客户端访问服务器所连接的公网地址， 请求过来之后自然会通过路由找到内网所在的网卡。建立链接。 
9. 当程序从看的角度找不到问题， 就只能调试了， gdb打断点break到出问题的之前点， 然后run， next，排除问题。 而且很多知名的开源代码会经常使用基类指针的运行时多态， 如果不走运行调试分析， 很难理清整体的框架。 
10. 


### 设计问题
1. 如果我们还需要扩展的话， 需要将这些服务拆分，通过RPC框架注册成RPC服务， 然后供客户端请求， 有时候一个请求需要多个服务交互完成，这时候就需要zookeeper 注册中心，做服务的管理和统计， 让各个服务能够及时的响应和并保持一致性。但是很简单的一个问题， 贪多就不能精。 在这个找实习的节骨眼上， 做分布式明显不是自己现阶段的主要矛盾。
2.  线程数量和cpu核数量请保持一致， 这样避免不同核时候， 线程调用上下文调用过慢。 
3. 这个项目并没有使用数据库连接池， 因此每次查询都要创建数据库连接类效率比较低效。 
4.  用户和群的关系是多对多， 因此必须有一个中间表反应多对多之间的关系， 这里我们使用了一个id和组id联合主键的表作为中间表， 反应了用户和表的关系。 这是表的设计问题， 属于数据库开发工程师去做的， 你不用去设计出来这个表， 数据库开发工程师会给你理清里面的关系，给出表的定义。 
<<<<<<< HEAD
5.  创建群等操作没有ack， 我们可以自己添加。 
6. 不用担心json输入错误，我们调试时候是自己手动输入json， 而实际发送信息的是客户端， 到时候发送的格式是固定的。 
=======
5.  可以自学一些界面库或者前端的内容，将客户端从termail改成界面类型。 这是加分项。 
6. 注意这里我们没有使用redis的key-value这些内容，而是用了redis的一个发布订阅的小功能。因此如果面试时候被问到， 很可能关注点都不一样。 人家关注的数据如何保持一致性， 存储的原理。 我用的只有发布订阅设计模式。 
7. 数据库可以使用基于java语言开发的mycat中间件 ， 做分表分库， 主从复制，读写分离。 



# How use it ?


#   参考文献
1. https://blog.csdn.net/QIANGWEIYUAN/article/details/89023980
2. 服务器编程教程
3. https://blog.csdn.net/lihao19910921/article/details/81907795
4. https://blog.csdn.net/tjcyjd/article/details/69683360

#   版本记录