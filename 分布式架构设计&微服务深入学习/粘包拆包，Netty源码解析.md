###	粘包和拆包简介
	1.TCP网络编程中当我们读取或者发送消息的时候，都需要考虑TCP底层的粘包/拆包机制，这种情况无法避免。
	2.TCP是个“流”协议，就是没有界限的一串数据。TCP底层并不了解上层业务数据的具体含义，
	  它会根据TCP缓冲区的实际情况进行包的划分，所以在业务上认为，一个完整的包可能会被TCP拆分
	  成多个包进行发送，也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。
	3.粘包：可以通俗的理解为在缓冲区（无论发送数据还是接收数据我们都要经过操作系统的缓冲区）产生了数据
			堆积，多个请求的数据串在了一起。
	4.拆包：可以理解是是一个数据包的大小超过了操作系统中缓冲区的大小而产生了拆包的操作。
	5.带来的问题，粘包和拆包会导致 数据包拆分后的不完整 和 数据粘包重组之后的数据混乱的情况。
	
###	粘包和拆包的解决办法
	  因为底层协议无法理解上层业务的数据逻辑，所以底层无法保证数据不被拆分和重组，只能通过上层协议栈
	  进行处理，具体的处理的办法如下：
	  （1）固定数据包的大小，累计读取到长度和 等于 定长LEN的报文后，就认为读取到了完整的信息。
	  （2）将换行符作为消息的结束符。
	  （3）以特殊字符作为消息的结束符。
	  （4）通过消息头中定义的长度作为数据包的总长度。
	  
###	Netty中的粘包和拆包解决方案 Netty提供了4种解码器来解决，分别如下：
	1.固定长度的拆包器（FixedLengthFrameDecoder），应用层数据包的都拆分都固定长度的大小。
	2.行拆包器（LineBasedFrameDecoder）应用层数据包都以换行符作为分隔符，进行分割拆分。
	3.分隔符拆包器（DelimiterBasedFrameDecoder）应用层数据包都通过自定义的分隔符进行分割拆分。
	4.基于数据包长度的拆包器（LengthFieldBasedFrameDecoder）将应用层数据包的长度作为
	  接收端应用层数据包的拆分依据。按照应用层数据包的大小拆包。有个硬性要求就是应用
	  层协议中包含数据包的长度。

###	EventLoopGroup事件线程组源码解读
-	 netty为了发挥CPU多核的优势，会通过线程组的形式去创建事件处理线程（NioEventLoop），
	 该线程包括了：Selector（选择器，Channal注册），TaskQueue（任务队列）
	 1.new NioEventLoopGroup(1)；从线程组的构造方法为入口；
	 2.进行线程数设置，默认线程数量为处理器数*2，如果构造方法已经指定了数量就以指定值为线程数。
	 3.根据线程数量创建NioEventLoop，循环执行newChild()方法。根据线程数量创建代码路径如下：
	   io.netty.util.concurrent.MultithreadEventExecutorGroup#MultithreadEventExecutorGroup
	 4.创建TaskQueue和Selector：io.netty.channel.nio.NioEventLoop#NioEventLoop
			
###	Netty启动流程分析：	
	1.从bind()绑定方法进入到io.netty.bootstrap.AbstractBootstrap#doBind.
	2.创建通道和初始化并注册通道到当前线程组中：io.netty.bootstrap.AbstractBootstrap#initAndRegister
		（1）调用初始化：io.netty.bootstrap.ServerBootstrap#init
			1-1）赋值workGroup与服务端handler
			1-2）向通道添加初始化对象handler（p.addLast），在taskQueue执行的时候调用
			1-3）在initChannel方法中添加ServerBootstrapAcceptor的handler
		（2）调用注册：io.netty.channel.AbstractChannel.AbstractUnsafe#register
			   2-1）io.netty.util.concurrent.SingleThreadEventExecutor#execute		
				    执行NioEventLoop（可根据该对象的parent/children下的数量判断boss/work线程组）：			
			   2-2）判断当前线程是否属于NioEventLoop，此时并不是。
			   2-3）addTask(task)添加任务（注册通道的任务）到任务队列中，此时任务并不会被调用。
			   2-4）当前线程并不是NioEventLoop则执行if方法块中的代码，依次进入到：
					io.netty.util.concurrent.SingleThreadEventExecutor#doStartThread该方法中。
			   2-5）执行单线程事件执行器run方法：io.netty.channel.nio.NioEventLoop#run
			   2-6）该方法中有for(;;)这样的死循环，由于此时我们的通道并没有注册到Selector选择器上
				    所以会跳过switch方法块中的代码。
			   2-7）接下来会执行io.netty.channel.nio.NioEventLoop#processSelectedKeys方法获取事件，
				    由于当前通道都没有注册，自然也就不会存在事件。
			   2-8）接下来再执行：io.netty.util.concurrent.SingleThreadEventExecutor#runAllTasks(long)
					该方法会去执行EventLoop中的takeQueue中的任务，而当前就有一个（就是注册通道的任务）。
			   2-9）此时则执行execute传递进来的Runnable的run方法中的："回到当前步骤下（2-1）"
				    io.netty.channel.AbstractChannel.AbstractUnsafe#register0
						io.netty.channel.nio.AbstractNioChannel#doRegister执行到该方法后：
						1）selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
						   将channel注册到Selector上，
						   描述：javaChannel()：获取ServerSocketChannal对象，
						         eventLoop().unwrappedSelector()：获取Selector选择器
			  2-10）io.netty.channel.DefaultChannelPipeline#invokeHandlerAddedIfNeeded
					该方法主要向pipeline中添加handler主要是针对initChannel方法中
					当方法调用后则会回到 "1-2"的步骤。
			  2-11）此时会执行ch.eventLoop().execute方法：往BossGroup的pipline中添加
					ServerBootstrapAcceptor的handler，该handler的作用就是把BossGroup
					接收到的通道注册到WorkGroup中。
	3.”到达这里则说明服务端的注册和初始化已经全部完成了“
	  第二次执行到io.netty.util.concurrent.SingleThreadEventExecutor#execute
	  这个方法了，当判断boolean inEventLoop = inEventLoop()时，当前线程是EventLoop,为ture。
	  继续执行，层层跳出再次到达io.netty.util.concurrent.SingleThreadEventExecutor#runAllTasks(long)
	  方法的for(;;)循环体中，执行在 “2-11” 步ch.eventLoop().execute中添加Runnable的run方法。在此时
	  完成往pepline中添加ServerBootstrapAcceptor这个handler.
		（1）ServerBootstrapAcceptor这个handler的主要处理消息入站，在它的channelRead()方法中将我们自定义的
		     handler添加到pipeline中和将通道注册到workGroup线程组上。
	4.再次到达io.netty.util.concurrent.SingleThreadEventExecutor#runAllTasks(long)的方法
	  此次我们已经将通道注册到Selector选择器上了，但是并没有发生任何事件。执行 "2-7" 步时
	  processSelectedKeys为0。
	5.当客户端连接时，则会注册连接事件，并产生processSelectedKeys交给WorkGroup处理。  

###	 BossGroup/WorkGroup/消息入站源码分析
	1.BossGroup主要负责监听. workGroup负责消息处理。基于这个原则我们看下BossGroup如何将通道交给workGroup
	  的和处理消息读取的，也就是消息入站。
	2.基于上面Netty的Server端的启动源码分析我们知道服务端会有一个for(;;)死循环不断的处理任务，
	  这里会循环处理客户端与服务端交互时所产生的3个事件，Select(注册选择器)，
	  processSelectorKeys(将产生的SelectorKey注册到WorkGroup)。runAllTask(处理taskQueue任务队列的任务)。
	3.我们以io.netty.channel.nio.NioEventLoop#processSelectedKeys这方法为入口，看下服务端如何处理SelectorKey。
	  （1）io.netty.channel.nio.NioEventLoop#processSelectedKeysOptimized
	       进入方法后找到个方法，这个方法中有个for循环会根据SelectorKeys的数量进行循环处理，当没有客户端连接的
		   时候，这个size就会为0，当客户端连接的时候就会触发两个事件，一个Accepter(连接事件)，
		   ChannalRead(通道读取就绪事件)
	  （2）当产生事件之后再跟下代码看看如何处理SelectorKey：io.netty.channel.nio.NioEventLoop#processSelectedKey
	       在这个方法中我们看到通过readyOps这个变量来判断事件属于Read读取/Accepter连接事件，判断之后执行unsafe.read();
	  （3）io.netty.channel.nio.AbstractNioMessageChannel.NioMessageUnsafe#read
			1）在这个read方法中会读取数据并缓存起来doReadMessages(readBuf)，而当前属于连接事件我们观察这个readBuf变量
			   可以得知这个变量是（NioSocketChannal）。
			2）pipeline.fireChannelRead(readBuf.get(i));接下来会发布通道读取事件，此时观察这个pipeline中有一个
			   ServerBootStrapAdapter的一个handler,接下来主要就是将数据交给这个handler来处理。
			3）io.netty.channel.DefaultChannelPipeline#fireChannelRead
               调用通道读取方法 即处理器入站操作，在这个方法中调用handler读取方法（next.invokeChannelRead(m);）		
			4）io.netty.channel.AbstractChannelHandlerContext#invokeChannelRead(java.lang.Object)   
					io.netty.bootstrap.ServerBootstrap.ServerBootstrapAcceptor#channelRead
			   在这个方法中获得通道信息：final Channel child = (Channel) msg；
			   将服务端处理器添加到：pipeline中child.pipeline().addLast(childHandler);
			   将通道注册到workGroup线程组上：childGroup.register(child).addListener
			   
			   注意：接下来就是WorkGroup运行
			   
		    5）io.netty.channel.AbstractChannel.AbstractUnsafe#register   
				1）执行NioEventLoop：eventLoop.execute(new Runnable() {};)这次执行就不是之前的BossGroup了
                   而是WorkGroup。
				2）io.netty.util.concurrent.SingleThreadEventExecutor#execute添加注册任务，现在未执行。
				   判断当前线程是不是eventLoop，因为当前是workGroup第一次执行，所以为false。
				   添加到任务队列：addTask(task);
				   启动线程：startThread(); 
			6）执行线程：io.netty.util.concurrent.SingleThreadEventExecutor#doStartThread 
				执行单线程事件执行器run方法：SingleThreadEventExecutor.this.run();
				io.netty.channel.nio.NioEventLoop#run此时还未将通道注册注册到Selector选择器
				所以不会走switch代码块中的代码。而是往下执行：processSelectedKeys();runAllTasks();
			7）io.netty.util.concurrent.SingleThreadEventExecutor#runAllTasks(long)
				此时执行SocketChannal通道注册。
			8）	io.netty.channel.nio.AbstractNioChannel#doRegister将SocketChannel注册到Selector上。
			9）之前我们分析一旦客户端产生连接则会触发2个事件，第一个连接事件已经处理完毕，接下来是读取事件。
				1）io.netty.channel.nio.NioEventLoop#processSelectedKeysOptimized
					处理SelectedKey：processSelectedKey(k, (AbstractNioChannel) a);
					此时readyOps = 1为读取事件：int readyOps = k.readyOps()；
					读取数据：unsafe.read();我们进入这个方法。
				2）final ChannelPipeline pipeline = pipeline();	获取pipeline，此时观察pipeline已经添加了我们
				   自定义的handler.
				   byteBuf = allocHandle.allocate(allocator);数据读取；
				   pipeline.fireChannelRead(readBuf.get(i));发布通道读取事件
				3）io.netty.channel.AbstractChannelHandlerContext#invokeChannelRead
				   调用handler读取方法：next.invokeChannelRead(m);
				4）io.netty.channel.AbstractChannelHandlerContext#fireChannelRead
					调用通道读取-查找入站handler：invokeChannelRead(findContextInbound(MASK_CHANNEL_READ), msg);	
				5)io.netty.channel.AbstractChannelHandlerContext#invokeChannelRead(java.lang.Object)
					执行该方法下的：((ChannelInboundHandler) handler()).channelRead(this, msg);
					此时进入的是我们自定义的消息入栈处理handler.
			10)消息入栈执行流程解析完毕。