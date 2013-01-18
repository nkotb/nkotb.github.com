#为zookeeper添加新命令测试总结
##概述
	因为项目需要，最近看了下zk client与server的命令传输实现，并试着添加一个新的zk命令，以下是这个测试的一些总结。

##源码分析：
###client端
	Zk client与zk server通信遵循一定的包格式，请求包头部为RequestHeader，请求包体为跟操作类型有关的XRequest对象，响应包头对象为ReplyHeader，响应包体对象为跟操作类型有关的XResponse对象，通过操作代码OpCode来控制server执行一些操作。这些操作码定义在ZooDefs类中，请求会通过zk连接对象的submitRequest发送给server，并接收响应的结果。ZooKeeperMain控制client端启动console，它根据console中输入的命令调用ZooKeeper类中对应的方法，生成通信包发送给server，并接受回应。

 

###server端
	zk server通过submitRequest函数（这个函数虽然与client端的发送函数同名，但实现不同）接收包，请求包被存入Request对象中，各种操作码按ZooDefs中约定的解析，Request对象将通过FinalRequestProcessor类的processRequest函数进行包处理，生成响应头ReplyHeader，根据操作码生成包体Response对象。响应会通过zk连接对象的sendResponse方法发送给client。

	Zk server是树状管理数据的，类似于一个小的文件系统，常用的各类操作也就是对棵树进行操作，这些操作可以在DataTree类中找到对应的实现。

##添加新命令
下面我们来看看怎样为zk增加新命令

	1.processZKCmd中首先会对命令做检查看是否在commandMap表中，如果不在就认为命令非法，因此我们要首先在ZooKeeperMain中增加一个命令到commandMap中

		commandMap.put(...)
	2,然后需要在processZKCmd中增加对命令的判断分支，以触发zk做对应的操作

		else if (cmd.equals("new cmd") 

		Zk.newCmd（。。。）

		。。。
	3.再到ZooKeeper中增加对应newCmd的实现，多数实现都有是否采用默认watcher和提供自定义watcher的实现，这里我们也要提供多种实现

		public byte[] newCmd(final String path, Watcher watcher, Stat stat)

       		throws KeeperException, InterruptedException {

       		...

       		h.setType(ZooDefs.OpCode.newCmd);

		}

		public byte[] newCmd(String path, boolean watch, Stat stat)

       		throws KeeperException, InterruptedException {

       		...

		}
 

	4.newCmd实际上是通过ZooDefs.OpCode.newCmd来告诉server该执行哪个操作的。所以我们需要到ZooDefs的OpCode 接口中增加这个属性。
		public class ZooDefs {

     		public interface OpCode {

      			...

         		public final int newCmd = x;

      		...

	5.client端基本完成，下面是server端。

	6.server接到命令包后会检查里面的OpCode是否合法，这通过Request类中的isValid验证，需要在这里加入对newCmd的判断。

	7.接下来通过FinalRequestProcessor中的processRequest函数解析对应的OpCode并调用对应的函数

		case OpCode.newCmd:
	
	8.具体生成什么样的Response对象，client用哪种response对象接收，完全取决于命令的类型，这部分也可以自行实现，这里就不说了。

	9.至此基本上就差不多了