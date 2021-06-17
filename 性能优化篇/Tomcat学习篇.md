### 浏览器访问服务器的流程
	1.用户发起请求，根据url或者链接，客户端发起TCP链接请求;
	2.服务器接收请求并创建连接，建立连接通道；
	3.客户端生成Http格式请求包，发送请求数据包到服务器；
	4.服务器解析Http请求数据包，并执行请求，生成HTTP格式请求数据包，发送响应数据包；
	5.客户端解析Http数据包（静态资源/动态资源）展现到客户端
	小结：
		浏览器访问服务器使用的是Http协议，Http是应用层协议，用于定义数据通信的格式，具体的数
		据传输使用的是TCP/IP协议。
		
### Tomcat基本主体
	1.servlet规范：Servlet接口和Servlet容器这一整套内容叫作Servlet规范，主要是将Servlet接口放进Servlet容器中。
	2.Tomcat的主要身份：
		（1）HTTP服务器：主要是与客服端进行交互，接收客服端的请求。
		（2）Servlet容器：根据请求在容器中定位能够处理请求的Servlet，然后由能够处理请求的servlet调用业务类。
	3.具体的执行流程	
		（1）Http服务器会生成Http通信格式数据包以SetvletRequest对象封装；
		（2）将请求传递给Servlet容器，由Servlet容器根据请求信息去调用具体的servlet；
		（3）如果Servvlet未被初始化，则通过反射机制创建Servlet，并通过init方法完成实例化；
		（4）调用该Servlet的service处理方法完成请求处理，将请求封装成ServletResponse对象；
		（5）把ServletResponse对象传递给HTTP服务器，由Http服务器相应到客户端；
	4.Tomcat设计了两个核心组件：连接器（Connector）和容器（Container）来完成Tomcat的两个核心功能。
		（1）连接器组件：负责和客户端建立连接，处理Socket连接，负责网络字节流与Request和Response对象的转化；
		（2）容器，负责内部处理：加载和管理Servlet，以及具体处理Request请求；
		
### Tomcat连接器Coyote
	1.Coyote是Tomcat中连接器的组件名称, 是对外的接口。客户端通过Coyote与服务器建立连接、发送请求并接受响应。	
		（1）Coyote封装了底层的通信协议，（Socket请求及响应处理）
		（2）Coyote让容器组件和具体协议和IO操作完全解耦。
		（3）Coyote将Socket输入转换封装为Request对象，封装后交由Catalina容器进行处理，处
			 理请求完成后, Catalina通过Coyote提供的Response对象将结果写入输出流。
		（4）Coyote负责的是具体协议（应用层）和IO（传输层）的相关内容。
		（5）Tomcat默认采用Http1.1协议，默认采用的I/O方式为BIO，之后改为NIO。

### Coyote组件及作用
	1.Endpoint(终端)主要是对TCP/IP的实现，处理接收和发送Socket。
	2.Processor是对应用层（Http协议）的处理，接收来自终端的Socket，解析Socket的字节流并封装成Request和Response原生对象，
	  并交给Adpter处理提交到容器。
	3.protocolHandler:Coyote协议接口，通过EndPoint和Processor实现针对具体的协议的处理能力，Tomcat按照协议和I/O提供了
	  6个实现类：AjpNioProtocol，AjpAprProtocol，AjpNio2Protocol，Http11NioProtocol，Http11Nio2Protocol，Http11AprProtocol。
	4.Adapter：由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了自己的Request类来封装这些请求信息。
	  ProtocolHandler接口负责解析请求并生成Request类。但是这个Request对象不是标准的ServletRequest，不能用Request作为参数
	  来调用容器。Tomcat设计者的解决方案是引CoyoteAdapter，这是适配器模式的经典运用，连接器调用CoyoteAdapter的Sevice方法法，
	  传回的是Tomcat Request对象，CoyoteAdapter负责将Tomcat Request转成ServletRequest，再调用容器。
	  
### Tomcat Servlet容器 Catalina	
	1.Tomcat是⼀个由一系列可配置（conf/server.xml）的组件构成的Web容器，而Catalina是Tomcat的servlet容器。
	  从另一个角度来说，Tomcat本质上就是Servlet容器，因为Catalina才是Tomcat的核心，其他模块都是为Catalina提供支撑的。
	2.Tomcat启动的时候会初始化contaline实例，Catalina实例通过加载server.xml完成其他实例的创建，创建并管理一个Server，
	  Server创建并管理多个服务，每个服务可以有多个Connector和一个个Container。  
		（1）Catalina（整体实例）：负责解析Tomcat的配置文件（server.xml）, 以此来创建服务器Server组件并进行管理。
		（2）Server：服务器表示整个Catalina（容器） Servlet容器以及其它组件，负责组装并启动Servlaet引擎,Tomcat连接器。
			 Server通过实现Lifecycle接口，提供了种优雅的启动和关闭整个系统的方式。
		（3）Service服务是Server内部的组件，一个Server包含多个Service。它将若干个Connector组件绑定到一个Container（容器）。
		（4）Connector连接器：用于监听端口；将所有的连接器绑定到容器（Container）,用于处理请求。
		（5）Container：容器，负责处理用户的servlet请求，并返回对象给web用户的模块；
		
### Container 组件的具体结构
-	Container组件下有多种具体的组件，分别是Engine、Host、Context和Wrapper。这4种组件（容器）
-	是父子层级关系。Tomcat通过⼀种分层的架构，使得Servlet容器具有很好的灵活性。
	1.Engine：表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine，
	  但是一个引擎可包含多个Host。
	2.Host：代表一个虚拟主机/站点，可以给Tomcat配置多个虚拟主机地址，一个虚拟主机下可包含多个Context。
	3.Context：表示一个Web应用程序， 一个Web应用可包含多个Wrapper。
	4.Wrapper：表示一个Servlet，Wrapper作为容器中的最底层，不能包含子容器。
	小结：
		上述组件的配置其实就体现在conf/server.xml中。
		
### conf/server.xml文件配置
	1.<Server port="8005" shutdown="SHUTDOWN"> 根元素，port：关闭服务器的监听端口，shutdown：关闭服务器的指令字符串；
	2.<Listener className="org.apache.catalina.startup.VersionLoggerListener" /> 以日志形式输出服务器 、操作系统、JVM的版本信息；
	3.<Listener className="org.apache.catalina.core.AprLifecycleListener"SSLEngine="on" />加载（服务器启动）和销毁（服务器停止）APR。
	  如果找不到APR库，则会输出日志，并不影响Tomcat启动。
	4.<Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />  加载（服务器启动）和销毁（服务器停止）全局命名服务。
	5.<Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>在Context停止时重建Executor池中的线程，
	  以避免ThreadLocal相关的内存泄漏。
	6.<GlobalNamingResources><Resource/><GlobalNamingResources/>定义了全局命名服务.
	7.<Service name="Catalina"></Service>该标签用于创建 Service 实例，默认使用 org.apache.catalina.core.StandardService。
	  默认情况下，Tomcat 仅指定了Service 的名称， 值为 "Catalina"。Service 子标签为 ：
		（1）Listener 用于为Service添加生命周期监听器。
		（2）Executor 用于配置和添加Service 共享线程池。
		（3）Connector 用于配置Service 包含的链接器.
		（4）Engine 用于配置Service中链接器对应的Servlet，Engine 表示 Servlet 引擎。
	8.Connector标签	标签用于创建链接器实例，默认情况下，server.xml 配置了两个链接器，一个支持HTTP协议，一个支持AJP协议
	  大多数情况下，我们并不需要新增链接器配置，只是根据需要对已有链接器进行优化。
	9.Host 标签用于配置一个虚拟主机。
	10.Context 标签用于配置多个Web应用。
	
### 核心流程源码解读（Tomcat启动流程/Tomcat请求处理流程）
	1.Tomcat启动流程：(逐层调用)
		从stratup.sh脚本出发，调用BootStrap的main方法，由BootStrap执行init方法，调用load方法；
		org.apache.catalina.startup.Bootstrap#init() -> org.apache.catalina.startup.Bootstrap#load 通过反射调用cataline容器的Load方法
			org.apache.catalina.startup.Catalina#load() （创建Server，并执行Server的init方法）
				org.apache.catalina.core.StandardService#initInternal（创建service，执行Engian.init初始化方法，执行Excutor初始化方法）
				调用connector连接组件的init方法 -> 调用protocolHandler的init方法
					执行Host的Init方法 -> 执行context的init方法
	2.Tomcat请求处理流程：
		（1）先到coyote连接组件被Endpoint终端接受Socket请求(Http应用层处理)。
		（2）调用processor将Socket封装成Request对象（TCP传输层处理）.
		（3）将Request传递给coyoteAdapter组件，将Request对象封装成标准的ServletRequest对象。
		（4）调用Engian，匹配对应的虚拟主机/站点（Host）。
		（5）调用Host，匹配对应的Context（应用）。
		（6）调用Context匹配对应的Wrapper。
		（7）根据匹配的Wrapper获取对应的servlet，并构造FilterChain(拦截器链)。
		（8）执行对应的Filter和执行Servlet。
		小结：
			在Tomcat请求过程中，根据Mapper组件体系结构去逐层匹配获取对用的目标对象，
			MapeerElement对象封装（MappedHost/MapperdContext.MapperWrapper等映射对象）
		
### JVM 类加载机制剖析	
	1.基础回顾：Java类 —> 字节码文件 —> 字节码文件需要被加载到jvm内存当中（这就是类加载的过程）。
	2.JVM的类加载机制：类加载器有自己的体系，Jvm内置了3种类加载器，包括：引导类加载器、扩展类加载器、
	  系统类加载器，他们之间形成父子关系，通过Parent属性来定义这种关系，最终形成树形结构。
		（1）引导类加载器：加载java核心库java.*,比如rt.jar中的类，构造ExtClassLoader和AppClassLoader。
		（2）扩展类加载器：加载扩展库 JAVA_HOME/lib/ext目录下的jar中的类，如classpath中的jre ，
			 javax.*或者java.ext.dir指定位置中的类。
		（3）系统类加载器：默认的类加载器，搜索环境变量 classpath 中指明的路径。
	3.双亲委派机制：当加载某个类的时候首先会将改class层层传递给父加载器，纸质顶级加载器，如果找不到则往下
	  传递给子加载器，如果子加载器依旧无法加载，则会抛出类构造异常。
	  双亲委派机制的作用就是为了保证代码安全，也就是防止造成代码污染。
	  
### Tomcat 类加载机制剖析	  
	1.Tomcat类加载机制是没有完成遵守双亲委派机制的，因为加入tomcat的应用是有多份的，其中就有可能引入了相同的class资源。
	2.引导类加载器 和 扩展类加载器 的作用不变。
	3.系统类加载器正常情况下加载的是CLASSPATH下的类，但是Tomcat的启动脚本并未使用该变量，而是加载tomcat启动的类，比如bootstrap.jar，
	  通常在catalina.bat或者catalina.sh中指定。位于CATALINA_HOME/bin下。
	4.Common 通用类加载器加载Tomcat使用以及应用通用的那些类，位于CATALINA_HOME/lib下，比如servlet-api.jar。
	5.Catalina ClassLoader 用于加载服务器内部可用类，这些类应用程序不能访问。
	6.Shared ClassLoader 用于加载应用程序共享类，这些类服务器不会依赖。
	7.Webapp ClassLoader，每个应用程序都会有一个独立的Webapp ClassLoader，他用来加载本应用程序 /WEB-INF/classes 和 /WEB-INF/lib 下的类。
	
## Tomcat 对 Https 的支持及 Tomcat 性能优化策略
### Tomcat 对 HTTPS 的支持
	1.Http超文本传输协议，明文传输 ，传输不安全，https在传输数据的时候会对数据进行加密ssl协议。
	2.Https传输流程：
		（1）浏览器客户端将自己支持的一套加密规则传递给服务端。
		（2）服务端从中选出一套加密规则，并将当前网站的信息以证书的形式传递给客服端，
			 证书中包含了当前的网站的信息还有公钥，证书机构的信息。
		（3）客户端收到后会校验证书机构是否可信，如果可信地址栏中会出现一个锁头状的标志，如果证书不可信的话
		     则会提示一个证书不合法的提示，如果用户手工选择可信的话，客户端会产生一段随机字符，并用公钥对其
			 进行加密，然后传递给服务端。
		（4）客户端收到后会使用私钥进行解密，然后用密码解密握手信息，然后再使用私聊对握手信息进行加密。	 
		（5）之后所有的通信都使用之前浏览器生成的随机密码和对称加密算法进行加密。

### Tomcat对SSL的支持
	使用 JDK 中的 keytool 工具生成免费的秘钥库文件(证书)。
	<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
	 maxThreads="150" schema="https" secure="true" SSLEnabled="true">
	 <SSLHostConfig>
		<Certificate certificateKeystoreFile="/Users/yingdian/workspace/servers/apache-tomcat-8.5.50/conf/lagou.keystore" 
		certificateKeystorePassword="lagou123" type="RSA"/>
	 </SSLHostConfig>
	</Connector>	
	
### Tomcat 性能优化策略	
	1.系统性能的衡量指标，主要是响应时间和吞吐量。
		（1）响应时间：执行某个操作的耗时；
		（2) 吞吐量：系统在给定时间内能够支持的事务数量，单位为TPS（Transactions PerSecond的缩写，
			 也就是事务数/秒，一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。
	2.Tomcat优化从两个方向进行：
		（1）JVM虚拟机优化（优化内存模型）
		（2）Tomcat自身配置的优化（比如是否使用了共享线程池？IO模型？）
	3.Java 虚拟机的运行优化主要是内存分配和垃圾回收策略的优化：
		（1）内存直接影响服务的运行效率和吞吐量
		（2）垃圾回收机制会不同程度地导致程序运行中断（垃圾回收策略不同，垃圾回收次数和回收效率都是不同的）
	4. Java 虚拟机内存相关参数：
		（1）-server 启动Server，以服务端模式运行。
		（2）-Xms 最小堆内存，建议与-Xmx设置相同。
		（3）-Xmx 最大堆内存，一般设置为可用内存的百分之80.
		（4）-XX:MatespaceSize 元空间初始值。
		（5）-XX:MaxMatespaceSize 元空间最大内存。
		（5）-XX:NewRatio 年轻代与老年代的大小比值，默认为2，一般不需要修改。
		（6）-XX:SurvivoeRatio Eden区与Survivor区大小的比值，取值为整数，默认为8。
	5.参数调整示例：JAVA_OPTS="-server -Xms2048m -Xmx2048m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"	
	6.垃圾回收性能指标
		（1）吞吐量：工作时间（排除GC时间）占总时间的百分比， 工作时间并不仅是程序运行的时间，还包含内存分配时间。
		（2）暂停时间：由垃圾回收导致的应⽤程序停⽌响应次数/时间。	
	7.垃圾收集器：
		（1）串行收集器（Serial Collector）单线程执行所有的垃圾回收动作， 适用于单核CPU服务器。
		（2）并行收集器（Parallel Collector）以并行的方式执行年轻代的垃圾回收， 该方式可以显著降
			 低垃圾回收的开销(指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态)。
			 适用于多处理器或多线程硬件上运行的数据量较大的应用。
		（3）并发收集器（Concurrent Collector）以并发的方式执行全部分垃圾回收工作，以缩短垃圾回收的暂停时间。
			 适用于那些响应时间优先于吞吐量的应用， 因为该收集器虽然最小化了暂停时间(指用户线程与垃圾收集线程同时执行,
			 但不一定是并行的，可能会交替进行)， 但是会降低应⽤程序的性能。
		（4）CMS收集器（Concurrent Mark Sweep Collector）并发标记清除收集器， 适⽤于那些更愿意缩短垃圾回收暂停时间并且
			 负担的起与垃圾回收共享处理器资源的应用。
		（5）G1收集器（Garbage-First Garbage Collector）适用于大容量内存的多核服务器，可以在满足垃圾回收暂停时间目标的同时，
			 以最大可能性实现高吞吐量(JDK1.7之后)
	8.垃圾回收器参数：
		（1）-XX:+UseSerialGC 启用串行收集器。
		（2）-XX:+UseParallelGC 启用并行垃圾收集器，配置了该选项，那么 -XX:+UseParallelOldGC默认启动。
		（3）-XX:+UseParNewGC XX:+UseParNewGC 年轻代采用并行收集器，如果设置了-XX:+UseConcMarkSweepGC选项，自动启用。
		（4）-XX:ParallelGCThreads 年轻代及老年代垃圾回收使用的线程数。默认值依赖于JVM使用的CPU个数。
		（5）-XX:+UseConcMarkSweepGC（CMS）对于老年代，启用CMS垃圾收集器。 当并行收集器无法满足应用的延迟需
			 求时，推荐使用CMS或G1收集器。启用该选项后， -XX:+UseParNewGC自动启用。
		（6）-XX:+UseG1GC 启用G1收集器。 G1是服务器类型的收集器，用于多核、大内存的机器。
			 它在保持高吞吐量的情况下，大概率满足GC暂停时间的目标。	 
		小结：在bin/catalina.sh的脚本中 , 追加如下配置 :JAVA_OPTS="-XX:+UseConcMarkSweepGC"
	9.Tomcat 配置调优，调整tomcat线程池配置：
		1.根据业务类型（IO密集型业务/CPU密集型业务）和服务器硬件条件，动态的配置maxConnections（最大连接数）
		  maxThreads（最大线程数），acceptCount（最大队列等待数）
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	