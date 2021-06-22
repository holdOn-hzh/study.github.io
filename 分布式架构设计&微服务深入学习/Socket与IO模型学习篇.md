###  Socket
	1.Socket，套接字就是两台主机之间逻辑连接的端点。
	2.TCP/IP协议是传输层协议，主要解决数据如何在网络中传输，而HTTP是应用层协议，主要解决如何包装数据。
	3.Socket是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。
	4.它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议、本地主机的IP地址、
	  本地进程的协议端口、远程主机的IP地址、远程进程的协议端口。
	5.客户端（Socket）请求与服务器（ServerSocket）进行连接的时候，根据服务器的域名或者IP地址，加上端口号
	 (范围：0-65536)，打开一个套接字。当服务器接受连接后，服务器和客户端之间的通信就像输入输出流一样进行操作。  

### I/O模型
	1.I/O 模型就是用什么样的通道进行数据的发送和接收，很大程度上决定了程序通信的性能。
	2. Java共支持3种网络编程模型/IO 模式：BIO(同步并阻塞)、NIO(同步非阻塞)、AIO(异步非阻塞)。
		（1）阻塞与非阻塞：主要指的是访问IO的线程是否会阻塞（或处于等待）。
		（2）主要是指的数据的请求方式，同步和异步是指访问数据的一种机制。是否同步等待，还是有事件进行回调通知。
	3.BIO(同步并阻塞)服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，
	  如果这个连接不做任何事情会造成不必要的线程开销。而且当前连接没有数据可读就会造成阻塞，对性能是一个浪费。
	4.NIO（同步非阻塞）服务器实现模式为一个线程处理多个请求(连接)，即客户端发送的连接请求都会注册到多路复用器上，
	 多路复用器轮询到连接有 I/O 请求就进行处理。
	5. AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统
	   完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用。
			（1）Proactor 模式是一个消息异步通知的设计模式，Proactor 通知的不是就绪事件，而是操作完成事
				件，这也就是操作系统异步 IO 的主要模型。

### BIO、NIO、AIO 适用场景分析
	1. BIO(同步并阻塞) 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，
	   并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解。
	2. NIO(同步非阻塞) 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕
	  系统，服务器间通讯等。编程比较复杂，JDK1.4 开始支持。
	3.AIO(异步非阻塞) 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分
	 调用 OS 参与并发操作， 编程比较复杂，JDK7 开始支持。

### NIO编程		
	1. NIO 有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector(选择器) 
	2. NIO是 面向缓冲区编程的。数据读取到一个缓冲区中，需要时可在缓冲区中前后移动，这就增加了
	   处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。   
	3. Java NIO的非阻塞模式，线程从某通道发送请求或者读取数据，如果目前没有数据可用时，不会保持线程阻塞，
	   所以数据变的可以读取之前，该线程可以继续做其他的事情。 
	4.非阻塞写也是如此，线程请求写入数据到通道，不需要等待它完全写入，这个线程同时可以去做别的事情。
	  通俗理解：NIO 是可以做到用一个线程来处理多个操作的。
	5.假设有 10000 个请求过来,根据实际情况，可以分配50 或者 100 个线程来处理。
	  不像之前的阻塞 IO 那样，非得分配 10000 个。

### NIO和 BIO的比较
	1. BIO 以流的方式处理数据,而 NIO 以缓冲区的方式处理数据,缓冲区 I/O 的效率比流 I/O 高很多。
	2. BIO 是阻塞的，NIO则是非阻塞的
	3. BIO基于字节流和字符流进行操作，而 NIO 基于 Channel(通道)和 Buffer(缓冲区)进行操作，数据
	   总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择器)用于监听多个通道的
	   事件（比如：连接请求， 数据到达等），因此使用单个线程就可以监听多个客户端通道

###  Selector、Channel和Buffer的关系	
	1. 每个 channel 都会对应一个 Buffer。
	2. Selector 对应一个线程， 一个线程对应多个 channel(连接) 3. 每个 channel 都注册到 Selector选择器上。
	4. Selector不断轮询查看Channel上的事件, 事件是通道Channel非常重要的概念。
	5. Selector 会根据不同的事件，完成不同的处理操作。
	6. Buffer 就是一个内存块 ， 底层是有一个数组。
	7. 数据的读取写入是通过 Buffer, 这个和 BIO , BIO 中要么是输入流，或者是输出流, 不能双向，但是
	   NIO 的 Buffer 是可以读也可以写 , channel 是双向的。

###  缓冲区（Buffer）
	1.缓冲区本质上是个可以读写数据的内存块，可以理解成是一个数组，该对象提供了一组方法，
	  可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。
	  Channel 提供从网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer.	   
	2.在 NIO 中，Buffer是一个顶层父类，它是一个抽象类, 类的层级关系图,常用的缓冲区分别对应
	  byte,short, int, long,float,double,char 7种.  

### Channel
	1.常用的Channel实现类类 有：FileChannel,DatagramChannel,ServerSocketChannel和
	  SocketChannel。FileChannel用于文件的数据读写， DatagramChannel用于UDP的数据读
	  写，ServerSocketChannel 和SocketChannel 用于TCP的数据读写。【ServerSocketChanne
	  类似 ServerSocket , SocketChannel 类似 Socket】	   
	2. SocketChannel 与ServerSocketChannel类似 Socke和ServerSocket,可以完成客户端与服务端数据的通信工作.   

#  	ServerSocketChannel（服务端实现步骤）
	1. 打开一个服务端通道
		ServerSocketChannal serverChannal = ServerSocketChannal.open();
	2. 绑定对应的端口号
	   serverSocketChannal.bind(new InetSocketAddress(9999));
	3. 通道默认是阻塞的，需要设置为非阻塞
	   serverSocketChannal.configureBlocking(false);
	4. 检查是否有客户端连接 有客户端连接会返回对应的通道
	   SocketChannel accept = serverSocketChannal.accept();
	5. 获取客户端传递过来的数据,并把数据放在byteBuffer这个缓冲区中
	   ByteBuffer readBuffer = ByteBuffer.allocate(1024); 
	   int readSize = accept.read(readBuffer);
	6. 给客户端回写数据
	   ByteBuffer write = ByteBuffer.wrap("你好".getBytes(StandardCharsets.UTF_8));
	   accept.write(write);
	7. 释放资源
	   accept.close();

###	SocketChannel实现步骤
	1. 打开通道
	   SocketChannel socketChannel = SocketChannel.open();
	2. 设置连接IP和端口号
	   socketChannel.connect(new InetSocketAddress("127.0.0.1",9999));
	3. 写出数据
	   ByteBuffer wrap = ByteBuffer.wrap("老板好".getBytes(StandardCharsets.UTF_8));
	   socketChannel.write(wrap);
	4. 读取服务器写回的数据
	   ByteBuffer allocate = ByteBuffer.allocate(1024);
	   socketChannel.read(allocate);
	5. 释放资源
		socketChannel.close();

###	Selector (选择器)
	1.可以用一个线程处理多个的客户端连接，使用NIO的Selector(选择器). Selector 能够检测多个注册的服务端
	  通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个
	  单线程去管理多个通道，也就是管理多个连接和请求。
	2.只有在通道真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都
	  创建一个线程，不用去维护多个线程, 避免了多线程之间的上下文切换导致的开销。
	3.SelectionKey中定义的4种事件:
		（1）SelectionKey.OP_ACCEPT（接收连接继续事件）表示服务器监听到了客户连接，服务器
			 可以接收这个连接了。
		（2）SelectionKey.OP_CONNECT（连接就绪事件）表示客户端与服务器的连接已经建立成功。
		（3）SelectionKey.OP_READ （读就绪事件）表示通道中已经有了可读的数据，可以执行读操
		     作了（通道目前有数据，可以进行读操作了）
		（4）SelectionKey.OP_WRITE（写就绪事件），表示已经可以向通道写数据了
	4.常用方法：
		（1）SelectionKey.isAcceptable(): 是否是连接继续事件
		（2）SelectionKey.isConnectable(): 是否是连接就绪事件
		（3）SelectionKey.isReadable(): 是否是读就绪事件
		（4）SelectionKey.isWritable(): 是否是写就绪事件

###	Selector（服务端实现步骤）
	public static void main(String[] args) throws IOException {
	        //1.创建服务端连接通道
	        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
	        //2.绑定对应的端口
	        serverSocketChannel.bind(new InetSocketAddress(9999));
	        //3.设置通道为非阻塞
	        serverSocketChannel.configureBlocking(false);
	        //4.创建选择器，供后面通道注册
	        Selector selector = Selector.open();
	        /**
	         * 5.将服务端通断注册到Select选择器中,并指定连接准备就绪事件
	         * 描述：当注册了连接准备就绪后，Select就会扫描来自客户端的连接，
	         *      将连接添加到我们这个selector选择器中
	         */
	        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
	
	        //持续运行程序对事件进行一个处理
	        while (true) {
	            /**
	             * 6. 检查选择器是否有事件
	             * 我们传入的这个2000的这个参数是个时间参数，表示多少时间内获取到对应的事件
	             */
	            int select = selector.select(2000);
	            if (select <= 0) continue;
	
	            //7.获取对应的事件合集，对对其进行迭代处理
	            Set<SelectionKey> selectionKeys = selector.selectedKeys();
	            Iterator<SelectionKey> iterator = selectionKeys.iterator();
	            while (iterator.hasNext()) {
	                SelectionKey next = iterator.next();
	                // 8. 判断事件是否是客户端连接事件SelectionKey.isAcceptable()
	                if (next.isAcceptable()) {
	                    /**
	                     * 9.得到客户端通道，将其设置为非阻塞，将其注册到Selector选择器中并指定为读事件
	                     * 描述：1.我们将原本的接受客户端连接的步骤移动到这里。
	                     *      2.为什么是读事件？因为我们现在是处于服务端的角度，
	                     *        所以我们需要读取来自客户端的数据。
	                     */
	                    SocketChannel accept = serverSocketChannel.accept();
	                    accept.configureBlocking(false);
	                    accept.register(selector, SelectionKey.OP_READ);
	                }
	                if (next.isReadable()) {
	                    //10.判断当前事件是否属于读事件，
	                    //   如果是的话我们可以通过这个selectionKey对象获取对应的channal
	                    SocketChannel channel = (SocketChannel)next.channel();
	                    //11.获取客户端通道中的数据
	                    ByteBuffer buffer = ByteBuffer.allocate(1024);
	                    int read = channel.read(buffer);
	                    if (read>0) {
	                        System.out.println("收到客户端的数据" + 
	                        new String(buffer.array(), StandardCharsets.UTF_8));
	                        //12. 给客户端回写数据
	                        channel.write(ByteBuffer.wrap("你好".getBytes(StandardCharsets.UTF_8)));
	                    }
	                    channel.close();
	                }
	                //13.删除当前事件，防止二次处理
	                iterator.remove();
	            }
	        }
	    }




























​	  





