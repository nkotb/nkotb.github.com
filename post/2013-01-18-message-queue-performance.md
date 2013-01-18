#几款开源消息队列性能及易部署性比较
##概述
对activemq，metaq，zeromq这几款开源消息队列产品做了下性能及易部署性比较,相关数据如下：

 

##测试环境

	Cpu Intel(R) Xeon(R)  E5620  @ 2.40GHz 
	Mem 48G

##测试方法
测试采用单条20B的消息数据，发送端与接收端都部署在一台机器上，所有测试均采用pub/sub模式，所有消息队列都没有开启消息持久化功能

 

##性能数据
以下是单进程读/写时的性能


	mq种类 	   | 	读效率 	| 	写效率 	| 	易部署性
	---------- 	| ---------	| ---------	| ---------
	Activema 	| 	1w/s 	| 	8k/s 	| 	高
	metaq 		| 	10w/s 	| 	1.1k/s 	| 	低
	zeromq 		| 	147w/s 	| 	144w/s 	| 	中

 

总的来看：

* zeromq性能上以压倒性优势胜出；

* metaq因为是单进程写，写入性能未全部发挥，但其部署过于复杂；

* activemq的webui可以很直观的观察到mq的使用情况，十分人性化；

##性能测试详细说明
###Activemq
* 部署简单：Activemq 部署较为简单，部署jdk后下载bin包运行即可

* Web ui监控： activemq提供了webui监控mq的性能

* Activemq中提供了client使用的jar包，引入后可根据api编写java客户端

* 进行读写测试时写客户端采用每次写入1000个20B的消息，读客户端每次读1000个消息，记录写入与读取时间，为保证正确性多次取样取平均值

		读速度 			| 写速度
		99ms 			| 127ms
		97ms 			| 132ms
		89ms 			| 125ms
		92ms 			| 124ms
		89ms 			| 125ms
		93ms 			| T127ms
		平均读取速度1w/s 	| 平均写入速度8k/s

###Metaq
* 部署复杂：除了必须的jdk外，server重度依赖zookeeper，而客户端需要引入Jackson，Zkclient，Zookeeper，Metaq-client，Thrift中的java lib部分的共计十几个jar包才能正确编译

* 进行读写测试时写客户端采用每次写入5000个20B的消息，读客户端每次读10000个消息，记录写入与读取时间，为保证正确性多次取样取平均值

				读速度 					| 		写速度
		10000 msg,total time is 101ms 	| 	Send 5000 msg cost 4588 ms
		10000 msg,total time is 100ms 	| 	Send 5000 msg cost 4531 ms
		10000 msg,total time is 103ms 	| 	Send 5000 msg cost 4550 ms
		10000 msg,total time is 108ms 	| 	Send 5000 msg cost 4621 ms
		10000 msg,total time is 105ms 	| 	Send 5000 msg cost 4608 ms
		10000 msg,total time is 101ms 	| 	Send 5000 msg cost 4735 ms
		平均读取速度 10w/s 				| 	平均写入速度 1.1k /s 
 

###Zeromq
* zeromq是lib库，部署完成后可自行编写server和client，编译时指定-lzmq即可

* 进行读写测试时写客户端采用每次写入100w个20B的消息，读客户端每次读50000个消息，记录写入与读取时间，为保证正确性多次取样取平均值

 

					读速度		 		  	| 	写速度
		send 1000000 message cost 659 ms 	| recv 50000 message cost 36 ms
		send 1000000 message cost 597 ms 	| recv 50000 message cost 33 ms
		send 1000000 message cost 735 ms 	| recv 50000 message cost 34 ms
		send 1000000 message cost 727 ms 	| recv 50000 message cost 33 ms
		send 1000000 message cost 741 ms 	| recv 50000 message cost 33 ms
		send 1000000 message cost 798 ms 	| recv 50000 message cost 32 ms
		send 1000000 message cost 665 ms 	| recv 50000 message cost 34 ms
		平均读取速度 147w						| 平均写入速度 144w /s 
 

###后续工作
后续将测试多pub多sub模式的读写效率



