### Netty介绍
	1.Netty的作用就是为了简化NIO的开发，并且做了额外的扩展，如：断连重连、网络闪断、半包读写、
	失败缓存、网络拥塞和异常流的处理。
	2.解决JDK NIO的Bug：Epoll Bug，它会导致Selector空轮询，最终导致CPU100%。直到
	  JDK 1.7版本该问题仍旧存在，没有被根本解决。
	3.Netty 的强大之处：零拷贝、可拓展事件模型；支持 TCP、UDP、HTTP、WebSocket 等协议；
	  提供安全传输、压缩、大文件传输、编解码支持等等。
	4.提供阻塞和非阻塞的 Socket；提供灵活可拓展的事件模型；提供高度可定制的线程模型。
	5.具备更高的性能和更大的吞吐量，使用零拷贝技术最小化不必要的内存复制，减少资源的消耗。
	6.提供安全传输特性。
	7.支持多种主流协议；预置多种编解码功能，支持用户开发私有协议。

###	线程模型基本介绍
	1.不同的线程模式，对程序的性能有很大影响，在学习Netty线程模式之前，首先讲解下各个线程模
      式，最后看看 Netty 线程模型有什么优越性.目前存在的线程模型有：
		(1)传统阻塞 I/O 服务模型
		(2)Reactor模型；根据 Reactor 的数量和处理资源池线程的数量不同，有3种典型的实现
		   单 Reactor 单线程，单 Reactor 多线程，主从 Reactor 多线程。
	2.传统阻塞 I/O 服务模型：采用阻塞 IO 模式获取输入的数据, 每个连接都需要独立的
	  线程完成数据的输入 , 业务处理和数据返回工作。
	  
###	Reactor 模型
-	Reactor 模式，通过一个或多个输入同时传递给服务处理器的模式, 服务器端程序处理传入的多个
-	请求,并将它们同步分派到相应的处理线程， 因此 Reactor 模式也叫 Dispatcher模式. Reactor 模式使用
-	IO 复用监听事件, 收到事件后，分发给某个线程(进程), 这点就是网络服务器高并发处理关键。
	
	1.单Reactor单线程：
	  （1）Selector是可以实现应用程序通过一个阻塞对象监听多路连接请求
	  （2）Reactor 对象通过 Selector监控客户端请求事件，收到事件后通过 Dispatch 进行分发
	  （3）是建立连接请求事件，则由 Acceptor 通过 Accept 处理连接请求，然后创建一个 Handler 对
	       象处理连接完成后的后续业务处理。handler会完成Read→业务处理→Send的完整业务流程。
		小结：优点：模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成。
			  缺点：存在性能问题: 只有一个线程，无法完全发挥多核 CPU 的性能。
				    可靠性问题: 线程意外终止或者进入死循环，会导致整个系统通信模块不可用，不能接收和处
					理外部消息，造成节点故障。
	2.单Reactor多线程：
		（1）Reactor 对象通过 selector 监控客户端请求事件, 收到事件后，通过 dispatch 进行分发
			 如果建立连接请求, 则右 Acceptor 通过accept 处理连接请求。
		（2）如果不是连接请求，则由 reactor 分发调用连接对应的 handler 来处理。
		（3）handler 只负责响应事件，不做具体的业务处理, 通过 read 读取数据后，会分发给后面的
			 worker 线程池的某个线程处理业务
		（4）worker 线程池会分配独立线程完成真正的业务，并将结果返回给 handler
			 handler 收到响应后，通过 send 将结果返回给 client
		小结：优点（可以充分的利用多核 cpu 的处理能力）
			  缺点：多线程数据共享和访问比较复杂， reactor 处理所有的事件的监听和响应，在单线程运行， 在
					高并发场景容易出现性能瓶颈
	3.主从Reactor多线程		
		（1）Reactor 主线程 MainReactor 对象通过 select 监听客户端连接事件，收到事件后，通过
			 Acceptor 处理客户端连接事件。
		（2）当 Acceptor 处理完客户端连接事件之后（与客户端建立好 Socket 连接），MainReactor 将
			 连接分配给 SubReactor。（即：MainReactor 只负责监听客户端连接请求，和客户端建立连
			 接之后将连接交由 SubReactor 监听后面的 IO 事件。)
		（3）SubReactor 将连接加入到自己的连接队列进行监听，并创建 Handler 对各种事件进行处理
			 当连接上有新事件发生的时候，SubReactor 就会调用对应的 Handler 处理
		（4）Handler 通过 read 从连接上读取请求数据，将请求数据分发给 Worker 线程池进行业务处理
			 Worker 线程池会分配独立线程来完成真正的业务处理，并将处理结果返回给 Handler。
		（5）Handler 通过 send 向客户端发送响应数据
		（6）一个 MainReactor 可以对应多个 SubReactor，即一个 MainReactor 线程可以对应多个SubReactor 线程
		小结：优点：主从Reactor线程分工明确，主Reactor将连接转发给subReactor，subReactor响应客户端请求，无需与
			        MainReactor线程模型进行交互。多个SubReactor线程能够应对更高的并发请求。
			  缺点：编程复杂度较高。	
			  
###	Netty线程模型
-	Netty 的设计主要基于主从 Reactor 多线程模式，并做了一定的改进。	分别有3个版本。
	1.简单版Netty模型：分为：BossGroup，WorkGroup两个部分。
		（1）BossGroup 线程维护 Selector，ServerSocketChannel 注册到这个 Selector 上，只关注连接
			 建立请求事件（主 Reactor）。
		（2）当接收到来自客户端的连接建立请求事件的时候，通过 ServerSocketChannel.accept 方法获
		     得对应的 SocketChannel，并封装成 NioSocketChannel 注册到 WorkerGroup 线程中的
			 Selector，每个 Selector 运行在一个线程中（从 Reactor）。
		（3）当 WorkerGroup线程中的Selector监听到自己感兴趣的IO事件后，就调用Handler进行处理。
	2.进阶版Netty线程模型
		（1）有两组线程池：BossGroup 和 WorkerGroup，BossGroup 中的线程专门负责和客户端建立
			 连接，WorkerGroup 中的线程专门负责处理连接上的读写
		（2）BossGroup 和 WorkerGroup 含有多个不断循环的执行事件处理的线程，每个线程都包含一
			个 Selector，用于监听注册在其上的 Channel。
	3.详细版Netty模型
		（1）Netty 抽象出两组线程池：BossGroup 和 WorkerGroup，也可以叫做
			 BossNioEventLoopGroup 和 WorkerNioEventLoopGroup。每个线程池中都有
			 NioEventLoop 线程。BossGroup 中的线程专门负责和客户端建立连接，WorkerGroup 中的
			 线程专门负责处理连接上的读写。
		（2）BossGroup 和 WorkerGroup 的类型都是NioEventLoopGroup，NioEventLoopGroup相当于
			 一个事件循环组，这个组中含有多个事件循环，每个事件循环就是一个NioEventLoop。
		（3）NioEventLoop 表示一个不断循环的执行事件处理的线程，每个 NioEventLoop 都包含一个
			 Selector，用于监听注册在其上的 Socket 网络连接（Channel）
		（4）NioEventLoopGroup 可以含有多个线程，即可以含有多个 NioEventLoop。

###	核心API介绍
	1.ChannelHandler及其实现类（ChannalInboundHandler入站）（ChannalOutBoundHandler出站）（ChannalhandlerAdapter出/入站）
		（1）Netty开发中需要自定义一个 Handler 类去实现 ChannelHandle接口或其子接口或其实现类，然后
		（2）通过重写相应方法实现业务逻辑，一般都需要重写方法：
			public void channelActive(ChannelHandlerContext ctx)，通道就绪事件
			public void channelRead(ChannelHandlerContext ctx, Object msg)，通道读取数据事件
			public void channelReadComplete(ChannelHandlerContext ctx) ，数据读取完毕事件
			public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)，通道发生异常事件
	2.ChannelPipeline
		（1）ChannelPipeline是一个Handler的集合，它负责处理和拦截inbound或者outbound的事件和
			 操作，相当于一个贯穿 Netty 的责任链。
		（2）如果客户端和服务器的Handler是一样的，消息从客户端到服务端或者反过来，每个Inbound类型
			 或Outbound类型的Handler只会经过一次，混合类型的Handler（实现了Inbound和Outbound
			 的Handler）会经过两次。
		（3）准确的说ChannelPipeline中是一个ChannelHandlerContext,每个上下文对象
			 中有ChannelHandler. InboundHandler是按照Pipleline的加载顺序的顺序执行, 
			 OutboundHandler是按照Pipeline的加载顺序，逆序执行。
	3.ChannelHandlerContext
		（1）这是事件处理器上下文对象，Pipeline链中的实际处理节点。每个处理节点
			 ChannelHandlerContext 中包含一个具体的事件处理器ChannelHandler,同时
			 ChannelHandlerContext 中也绑定了对应的 ChannelPipeline和 Channel 的信息，方便对
			 ChannelHandler 进行调用。常用方法如下所示：
				ChannelFuture close()，关闭通道
				ChannelOutboundInvoker flush()，刷新
				ChannelFuture writeAndFlush(Object msg) ，将数据写到ChannelPipeline中当前
				ChannelHandler的下一个ChannelHandler开始处理（出站）
	4.ChannelOption
		（1）Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数。ChannelOption 是 Socket 的标
			 准参数，而非 Netty 独创的。常用的参数配置有：
		（2）ChannelOption.SO_BACKLOG
			1）对应 TCP/IP 协议 listen 函数中的 backlog 参数，用来初始化服务器可连接队列大小。服务端处理
			   客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户 端来的时候，服
			   务端将不能处理的客户端连接请求放在队列中等待处理，backlog 参数指定 了队列的大小。
			2）ChannelOption.SO_KEEPALIVE ，一直保持连接活动状态。该参数用于设置TCP连接，当设置该选
			   项以后，连接会测试链接的状态，这个选项用于可能长时间没有数据交流的连接。当设置该选项以
			   后，如果在两小时内没有数据的通信时，TCP会自动发送一个活动探测数据报文。
	5.ChannelFuture
		（1）表示 Channel 中异步 I/O 操作的结果，在 Netty 中所有的 I/O 操作都是异步的，I/O 的调用会直接返
			 回，调用者并不能立刻获得结果，但是可以通过 ChannelFuture 来获取 I/O 操作 的处理状态。
		（2）常用方法如下所示：
			 Channel channel()，返回当前正在进行 IO 操作的通道
			 ChannelFuture sync()，等待异步操作执行完毕,将异步改为同步
	6.EventLoopGroup和实现类NioEventLoopGroup 
		（1）EventLoopGroup 是一组EventLoop的抽象，Netty为了更好的利用多核CPU资源，一般会有多个
			 EventLoop同时工作，每个 EventLoop 维护着一个 Selector 实例。
		（2）BossEventLoopGroup 通常是一个单线程的 EventLoop，EventLoop 维护着一个注册了
			 ServerSocketChannel 的 Selector 实例，BossEventLoop 不断轮询 Selector 将连接事件分离出来， 通
			 常是 OP_ACCEPT 事件，然后将接收到的 SocketChannel 交给 WorkerEventLoopGroup，
			 WorkerEventLoopGroup 会由 next 选择其中一个 EventLoopGroup 来将这个 SocketChannel 注
			 册到其维护的 Selector 并对其后续的 IO 事件进行处理。	 
	7.ServerBootstrap和Bootstrap
		（1）ServerBootstrap 是 Netty 中的服务器端启动助手，通过它可以完成服务器端的各种配置；
			 Bootstrap 是 Netty 中的客户端启动助手，通过它可以完成客户端的各种配置。
	8.Unpooled类		 
		（2）这是 Netty 提供的一个专门用来操作缓冲区的工具类，常用方法如下所示：
		     public static ByteBuf copiedBuffer(CharSequence string, Charset charset)，通过给定的数据 
			 和字符编码返回一个 ByteBuf 对象（类似于 NIO 中的 ByteBuffer 对象）
			 
###	Netty服务端编写	
	1.创建bossGroup线程组: 处理网络事件--连接事件 线程数默认为: 2 * 处理器线程数 
	EventLoopGroup bossGroup = new NioEventLoopGroup(1); 
	2.创建workerGroup线程组: 处理网络事件--读写事件 2 * 处理器线程数 
	EventLoopGroup workerGroup = new NioEventLoopGroup(); 
	3.创建服务端启动助手 
	ServerBootstrap bootstrap = new ServerBootstrap(); 
	4.设置线程组 
	bootstrap.group(bossGroup, workerGroup);
	5.设置服务端通道实现;
	serverStrap.channel(NioServerSocketChannel.class) 
	6.参数设置-设置线程队列中等待 连接个数
	serverStrap.option(ChannelOption.SO_BACKLOG, 128) 
	7.参数设 置-设置活跃状态,child是设置workerGroup
	serverStrap.childOption(ChannelOption.SO_KEEPALIVE, Boolean.TRUE)
	8.创建一 个通道初始化对象
	serverStrap.childHandler(new ChannelInitializer<SocketChannel>() {
	@Override protected void initChannel(SocketChannel ch) throws Exception { 
	9.向pipeline中添加自定义业务处理
	handler ch.pipeline().addLast(new NettyServerHandle()); 
	} }); 
	10.启动服务端并绑定端口,同时将异步改为同步 
	ChannelFuture future = bootstrap.bind(9999).sync(); 
	11.关闭通道(并不是真正意义上的关闭,而是监听通道关闭状态)和关闭连接池 
	future.channel().closeFuture().sync(); 
	bossGroup.shutdownGracefully(); 
	workerGroup.shutdownGracefully(); }		 
			
###	自定义服务端handle		
	1.创建对象并实现implements ChannelInboundHandler接口，重写其方法。
	2.主要的方法有channelActive（通道就绪事件）channelRead（通道读取事件），
	  channelReadComplete（读取完毕事件），exceptionCaught（异常发生事件）。
		
###	2 Netty客户端编写		
	1. 创建线程组 
	EventLoopGroup group = new NioEventLoopGroup(); 
	2. 创建客户端启动助手 
	Bootstrap bootstrap = new Bootstrap(); 
	3. 设置线程组 
	bootstrap.group(group) 
	4. 设置服务端通道实现为NIO
	bootstrap.channel(NioSocketChannel.class) 
	5. 创建一个通 道初始化对象
	bootstrap.handler(new ChannelInitializer<SocketChannel>() {
	@Override protected void initChannel(SocketChannel ch) throws Exception { 
	6. 向pipeline中添加自定义业务处理handler 
	ch.pipeline().addLast(new NettyClientHandle()); }}); 
	7. 启动客户端, 等待连接服务端, 同时将异步改为同步 
	ChannelFuture future = bootstrap.connect("127.0.0.1", 9999).sync(); 
	8. 关闭通道和关闭连接池 
	future.channel().closeFuture().sync(); 
	group.shutdownGracefully();
	
###	Netty异步模型	
	1.异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调
	  用的组件在完成后，通过状态、通知和回调来通知调用者。
	2.Netty 中的 I/O 操作是异步的，包括 Bind、Write、Connect 等操作会简单的返回一个ChannelFuture。
	  调用者并不能立刻获得结果，而是通过 Future-Listener 机制，用户可以方便的主动获
	  取或者通过通知机制获得IO 操作结果. Netty 的异步模型是建立在 future 和 callback 的之上的。
	3.callback就是回调，Future表示异步的执行结果, 可以通过它提供的方法来检测执行是否完成，
	  ChannelFuture 是他的一个子接口. ChannelFuture是一个接口,可以添加监听器，当监听的事件发生时，
	  就会通知到监听器。  
	4. Future-Listener 机制：给Future添加监听器,监听操作结果。
		代码实现：
		ChannelFuture future = bootstrap.bind(9999);
        future.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                if (future.isSuccess()) {
                    System.out.println("端口绑定成功!");
                } else {
                    System.out.println("端口绑定失败!");
                }
            }
        });
	  
###	Netty编解码器	  
	1.java的编解码回顾：
		（1）编码（Encode）称为序列化，它将对象序列化为字节数组，用于网络传输、数据持久化或者其它用途。
		（2）解码（Decode）称为反序列化，它把从网络、磁盘等读取的字节数组还原成原始对象（通常是原
			 始对象的拷贝），以方便后续的业务逻辑操作。
	2.在网络应用中需要实现某种编解码器，将原始字节数据与自定义的消息对象进行互相转换。网络
	  中都是以字节码的数据形式来传输数据的，服务器编码数据后发送到客户端，客户端需要对数据进行解码。
	3.对于Netty而言，编解码器由两部分组成：编码器、解码器。
		（1）解码器：负责将消息从字节或其他序列形式转成指定的消息对象。
		（2）编码器：将消息对象转成字节或其他序列形式在网络上传输。
	4.Netty 的编（解）码器实现了 ChannelHandlerAdapter，也是一种特殊的 ChannelHandler，所
	  以依赖于 ChannelPipeline，可以将多个编（解）码器链接在一起，以实现复杂的转换逻辑。
	  Netty里面的编解码： 解码器：负责处理“入站 InboundHandler”数据。 编码器：负责“出站
	  OutboundHandler” 数据。	
	5.解码器(Decoder)
		（1）解码器负责 解码“入站”数据从一种格式到另一种格式，解码器处理入站数据是抽象
			 ChannelInboundHandler的实现。
		（2）需要将解码器放在ChannelPipeline中。
		（3）对于解码器，Netty中主要提供了抽象基类ByteToMessageDecoder和MessageToMessageDecoder	
		（4）ByteToMessageDecoder（用于将字节转为消息）MessageToMessageDecoder（用于从一种消息解码为另外一种消息）
	6.编码器(Encoder)	
		1.Netty提供了对应的编码器实现MessageToByteEncoder和MessageToMessageEncoder，二者都实现
		  ChannelOutboundHandler接口。
		2.MessageToByteEncoder: 将消息转化成字节，MessageToMessageEncoder: 用于从一种消息编码为
		  另外一种消息（例如POJO到POJO）
	7.编码解码器Codec	 
		1.编码解码器: 同时具有编码与解码功能，特点同时实现了ChannelInboundHandler和
		  ChannelOutboundHandler接口，因此在数据输入和输出时都能进行处理。
		2.Netty提供提供了一个ChannelDuplexHandler适配器类,编码解码器的抽象基类
		  ByteToMessageCodec ,MessageToMessageCodec都继承与此类。
		3.只要实现了上述的两个接口的其中一个，重写encode/decode可处理编解码工作。
		
		
		
		
		
		
		
		
		
		
		
		
		
		
