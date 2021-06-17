## SpringBoot

### 约定优于配置思想
	又称按约定编程，是一种软件设计规范。本质上是对系统、类库或框架中一些东西假定一个大众化合理的默认值(缺省值)。
	只要不偏离默认配置，就不需要额外配置；

### SpringBoot基本概念
	1.springBoot是一种快速启动Spring的一种轻量级框架，可以让我们使用Spring的强大功能，又不需要像Spring那样进行大量配置；
	2.SpringBoot是基于Spring4.0版本设计，它解决了依赖包的一个冲突问题还有引用不稳定的问题；
	
### SpringBoot的相关特性
- 	springBootStarter（起步依赖） springBoot将工程中所依赖的坐标进行了打包，并在打包的过程中解决依赖版本冲突和引用不稳定问题；
	也就是说本来我们需要引入10个依赖的坐标，但是springBoot将我们需要的这10个依赖坐标打包在了一起，我们只需要引用一个坐标即可，而我们无需记忆多个依赖坐标；
-	JavaConfig配置方式，由于我们是基于Spring4.0设计那自然就保留了这种配置方式，可以让我们基于Servlet3.1规范完全舍弃web.xml和其他xml配置；
	通过@Configuation注解将一个普通类定义为配置类，通过方法创建对象并在方法上添加@Bean注解将当前对象添加到IOC容器中；
- 	自动配置，SpringBoot在启动过程中会根据我们添加的jar包依赖，默认需要的配置类创建出来并放在IOC容器中；
-	部署更简单，SpringBoot默认内置了3种Servlet容器，分别是：Tomcat,Jetty,undertow。以前我们部署需要将工程打成war包并放在Tomcat容器上，而现在SpringBoot项目是
	打成jar包，只需要一个简单的java运行环境既可以启动；
	
### 热部署	
- 	热部署（devtoos插件），相对于冷部署（在重新部署过程中除了加载本地class文件外还会将第三方依赖重新加载）热部署只会加载我们本地的class文件；
-	热部署原理是：是对classpath资源进行一个监控，而当我们本地的classpath资源产生变化的时候则会触发RestartClassLoad类加载对我们本地的classpath资源进行重新加载；
	在我们的项目中classpath资源可以分为两个部分，一个是不变资源（第三方jar包的class资源），一个是可变资源（本地class资源）；当引入了热部署插件后则会通过两个类加载
	Base classLoader 和 RestartClassLoader进行类资源加载，BaseClassLoader加载的是第三方class，而RestartClassLoader加载的是我们本地的class。（注：在不引用插件的情况下使用默认类加载器）
	所以当我们的本地的class资源文件产生变化的时候就是通过RestartClassLoader类加载器重新加载class文件；
-	热部署排除资源配置方式：spring.devtools.restart.exclude=static/**,public/**

### 全局配置文件	
-	全局配置文件格式可支持application.properties（KV形式配置） application.yaml（json超文集格式）
-	全局配置文件的作用就是用来修改默认配置的，比如Servlet端口，servlet访问路径等等；
-	SpringBoot项目中可支持在4个地方中放置配置文件：./config/（根目录，优先级最高）;/(项目根目录下)；/resources/(资源文件下)；resources/config/(资源文件下的config包下)；
	其中./config/位置的优先级是最高的，在这4个不同的配置文件位置下如果同时配置的话：
	1.同属性配置默认取优先级最高的配置的文件，不会进行属性覆盖；
	2.配置了不同属性的情况下，所有的配置属性都会被执行；
	3.application.properties和application.yml同时配置的话：
		2.4.0之前版本优先级properties>yaml；
		2.4.0的版本，优先级yaml>properties
		如果想要使用之前的配置（2.3）：spring.config.use-legacy-processing = true
	4.如果配置文件的名称不想使用默认的application的话可在加载jar包时通过命令指定名称；java -jar myproject.jar --spring.config.name=myproject
-	application.properties中的配置，springBoot会通过@ConfiguationProperties注解将配置文件的属性注入给ServiceProperties这个对象；
-	注入形式可以分为单属性注入（@Value），批量注入（@Configuationproperties），第三方bean注入（在Bean创建方法上添加@ConfiguationProperties）,
	PropertySource("classpath:/jdbc.properties")指定外部属性文件。在类上添加;
-	支持松散绑定，也就是映射字段不需要完全匹配，比如羊肉串模式、驼峰命名，下划线模式；

### SpringBoot日志
-	通常情况下，日志是由一个抽象层+实现层的组合来搭建的。日志-抽象层有：JCL、SLF4J、jboss-logging；日志实现层：jul、log4j、logback、log4j2；
-	Spring 框架选择使用了 JCL 作为默认日志输出。而 Spring Boot 默认选择了 SLF4J 结合 LogBack；
-	在使用Slf4J + 切换日志实现层的时候，需要引入一个适配器依赖，因为在实现层的日志框架是不知道有Slf4J这个抽象层的，所以需要适配；	
-	统一日志框架：像我们当前项目中引入了自己的日志框架，但是我们还引入了第三方的框架，而这些第三方框架也有自己的日志框架，比如Spring（JCL），hibbnate（JossLogging）;
	日志框架统一方案：先排除第三方框架的日志依赖（<exclusions>），再导入日志适配转换包，将第三方日志依赖转换为我们当前系统中的日志框架；
		
### SpringBoot源码-依赖管理
	问题1：为什么引入依赖（dependency）时不需要指定依赖版本？
	spring-boot-starter-parent 通过继承 spring-boot-dependencies 从而实现了SpringBoot的版本依
	赖管理,所以我们的SpringBoot工程继承spring-boot-starter-parent后已经具备版本锁定等配置了,这也
	就是在 Spring Boot 项目中部分依赖不需要写版本号的原因
	
	问题2：spring-boot-starter-parent父依赖启动器的主要作用是进行版本统一管理，那么项目运行依赖的JAR包是从何而来的？
	依赖启动器的主要作用是打包了开发场景所需的底层所有依赖（基于依赖传递，当前项目也存在对应的依赖jar包）
	正是如此，在pom.xml中引入依赖启动器时，就可以实现对应的场景开发，而不需要额外导入依赖文件等。
	当然，这些引入的依赖文件的版本号还是由spring-boot-starter-parent父依赖进行的统一管理。
	
### SpringBoot源码-自动配置
	启动类上的@SpringBootApplication注解基本理解：
	1.该注解是一个组合注解，内部主要有（@SpringBootConfiguration（配置注解），@EnableAutoconfiguration(开启自动配置)，@ComponentScan（扫描类注解））
		1.@SpringBootConfiguration是对@Configuration配置注解的封装；
		2.@EnableAutoconfiguration里面也是个组合注解：
				1.@AutoConfigurationPackage（向IOC容器中注册了一个BasePackges的Bean，并将当前注解下的包名注册下来，供后续使用）；
				2.@Import(AutoConfigurationImportSelector.class)这个注册器：AutoconfigurationImportSelector主要是把所有符合@Configuration的配置bean
				  注册到SpringBoot的IOC容器（ApplicationContext）中；
				3.@Import主要的作用就是将某一个特定组件注册到IOC容器中；
				4.Aware接口一般都有一个回调方法，而这个Aware接口在调用的时候就会传入某种资源，AutoconfigurationImportSelector组件就是现实了：BeanFactoryAware,
				  ResourceLoaderAware，BeanClassLoaderAware，EnvironmentAware这4个接口，拿到资源后委托到本地；
				5.由于AutoconfigImportSelector实现了DeferredImportSelector对象那么则会触发它的一个子类叫做DeferredImportSelectorGropping下的getImports方法，
				  而该方法 会调用AotoConfigImportSelector下的process方法，而在该方法中会去获取所有jar包MATE-INFO下的spring.factories文件中以EnableAutoconfiguration
				  为key的配置类全限定类名；获取完成后并对不需要被加载的配置类进行排除，排除步骤如下： 
					1.去除重复配置 
					2.判断是否在启动类上的@SpringBOotApplication注解中有添加exclude属性中是否手动指定了需要排除的配置类，
					  如果有则去除。注意：如该指定配置类不是自动配置类则会抛出异常； 
					3.根据配置类上的@Conditional为前缀的注解判断，比如ConditionalOnClass中配置的Class去判定该配置类是否需要被加载，也就是说这个配置类是否需要加载和
					  我引入的依赖是有关系的，如果注解中配置的class不存在，则说明没有引入该依赖，该配置类不需要被加载； 
					4.process方法中做的事情就是收集了需要要自动配置的bean定义信息， 在run方法被执行的在初始化这些bean到IOC容器中； 
					5.在DeferredImportSelectorGroping中的getImports方法还有一个selectImports方法，该方法中主要做了两件事，一是再次对配置进行过滤判断是否需要排除，
					  二是根据配置类上的 @Order注解对配置类进行一个排序；
		3.@ComponentScan配置扫描路径，在该注解中并没有指定某个路径，只是借助excludeFilters将TypeExcludeFillter及FilterType这两个类进行排除；
		  而当没有指定路径的情况下，就会默认当前对象的路径及其子路径。这也是为什么SpringBoot只会扫描启动类及其子包的对象；
		小结：
			  @EnableAutoconfiguration（自动配置类处理）和 @ComponentScan（本地项目下的配置了注解的bean对象）
			  第一个注解是收集了自动配置类的信息并注册到AutoconfigueationImportSelector中，第二个则是对本地项目
			  需要加载到IOC容器中的对象进行一个扫描（当前并没有实例化对象和配置类中的@Bean注解标注的对象）
			  
### SpringBoot执行流程源码刨析
	在启动类中的run方法中有两件事：
	初始化SpringApplication 2.执行run方法 :return new SpringApplication(primarySources).run(args);
		1.new SpringApplication()初始化所做的事：
			1. this.webApplicationType = WebApplicationType.deduceFromClasspath(); 
			   推断应用类型，后面会根据类型初始化对应的环境。常用的一般都是servlet环境                        
			2. setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
			   加载并初始化所有jar报下的MATE-INFO下的Spring.factories文件中的以ApplicationContextInitializer为key对应的初始化器； 
			3. setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
			   获取所有jar包下的MATE-INFO的spring.factories文件中的监听器配置Bean对象的全限定类名并通过反射机制进行实例化；
			4.this.mainApplicationClass = deduceMainApplicationClass()对启动类的类名进行一个记录；
			小结：
				  ApplicationContextInitializer 是Spring框架的类, 这个类的主要目的就是 在 ConfigurableApplicationContext 调用refresh()方法之前，回调这个类的                        initialize方法。从而达到对配置的完善工作；
				  ApplicationListener 是spring的事件监听器，典型的观察者模式，通过ApplicationEvent 类和ApplicationListener 接口，可以实现对spring容器全生命周期的监听，当然也可以自定义监听事件；
		2.run方法中的执行流程：
			1. 获取并启动监听器：通过getspringFactoriesInstance方法加载和实例化SpringApplicationRunListener容器运行期监听器，
			   监听整个运行过程当产生特定事件的时候则传递给SpringApplicationListener处理。注：监听器是Bean与Bean之间通讯的一种手段；
			2. 构造应用上下文环境（jdk,maven,spring运行环境，配置等）：根据应用类型创建并配置相应的环境，根据用户配置（dev,test,master等环境，四种配置文件位置），
			   配置environment系统环境，然后启动监听器，并加载系统配置文件，之前实例化的监听器中的ConfigFileApplicationListener就是加载项目配置文件的监听器。
			3. 初始化应用上下文：根据项目的应用类型：非web应用（NONE），SERVLET（Servlet的web应用），REACT（响应式web应用）去创建对应的上下文对象，比如一般我们都是
			   sevelet的web应用则根据默认配置类的全限定类名：AnnotationConfigServletWebServerApplicationContext创建上下文对象；然后对其进行初始化；
			   AnnotationConfigServletWebServerApplicationContext对象继承了GenericWebApplicationContext这个容器对象，所以在初始化上下文对象的时候会
			   调用这个对象的构造器，在调用的时候就会创建对应的IOC容器DefaultListableBeanFactory并以属性的形式存在了上下文对象中；
			4. 刷新应用上下文前的准备阶段：对上下文对象属性的设置和bean的创建包括核心启动类，
				1.设置上下文属性，执行在准备阶段收集的ApplicationContextIntilizer初始化器；向各个监听器发布context已经准备好的事件；
				2.将启动类解析成BeanDefinition，并注册到beanFactory的beanDefinitionMap中；
			5. 刷新应用上下文：refreshContext调用的是AbstractApplicationContext（IOC容器）的refresh方法，然后在invokeBeanFactoryPostProcess方法中
			   对本地对象及自动配置bean实例化，步骤如下：
				1.obtainFreshBeanFactory获取我们之前创建的DefulListableBeanFactory，然后调用invokeBeanFactoryPostProcess；
				1.资源定位（@ConponScan扫描（解析主类的BeanDefinition获取basePackage的路径完成了定位的过程），@Import注解指定组件对象（主类上的@SpringBootApplication中使用））
				2.解析BeanDefinition对象：
					1.在拿到basepackge的路径后则会加载该路径下带有@Component注解的对象，将该对象加载成BeanDefinition;
					2.@Import注解引入组件，在processImports方法中会递归判断是否带有该注解，在有的情况下将其指定的注册器记录下来，然后调用defeedImportSelectorHandler.process方法。
					  在这个方法中会遍历调用所有的ImportSelector.getImports方法，@SpringBootApplication注解中的@Import注解配置的注册器是实现是了DeffeedImportSelector接口的，
					  所以AutoConfigImportSelector也就具有了该接口的process方法，则会在这个方法中实现自动配置类和配置类中被@Bean注解标注需要生成的Bean添加到BeanDefinitionMap中；
				3.注册BeanDefinition对象，将需要加载的class封装成BeanDefinition封装好后放进BeanFactory的BeanDefinitionMap属性中；
			6. 刷新应用上下文后的扩展接口：一个模板方法，是对上下文对象扩展用的；
			小结：
				  1.上下文个人理解：很多人会把容器理解为上下文，但是其实两者之间是一个持有和扩展的关系，上下文持有容器，并在容器之上丰富了一些内容，
				    我们在阅读源码的时候应该要把两者之间严格区分开来；
				  2.@Import注解说明：我们在阅读源码的过程一般以Enable开头的注解其内部一般都会使用@Import注解；
				    而该注解的作用就是将某一个组件注册到IOC容器当中；
				  
### 容易造成的误区：SpringBoot自动配置原理
	一般我们在被问及该问题的，一上来就会从启动类上的@SpringBootApplication注解讲起；但其实这种说法是不够全面的；
	我们应该结合启动类的run方法讲起，自动配置类被实例化的是因为我们在创建上下文对象后对其进行刷新阶段去调用Spring
	的refresh方法的中的invokeBeanFactoryPostProcess中完成的事；
	我们在此时就可以和@EnableAutoconfiguration衔接起来；

### starter机制
	1.SpringBoot中的starter是一种非常重要的机制，能够抛弃以前繁杂的配置，将其统一集成进starter，
	  应用者只需要在maven中引入starter依赖，SpringBoot就能自动扫描到要加载的信息并启动相应的默认配置。
	  starter让我们摆脱了各种依赖库的处理，需要配置各种信息的困扰。
	2.starter就是一个外部的项目，我们需要使用它的时候就可以在当前springboot项目中引入它。
	3.一般应用于独立于业务之外的配置工程，将这些公共/特有的配置放在一个特定的工程下，我们需要的时候就引用它，不需要额外的拷贝；
	实现方法如下：
	1.引入依赖：spring-boot-autoconfigure
	2.创建对应的javaBean对象，和创建一个配置类，并在配置类中生成该Bean，以@Bean注解标识，当该配置类被加载时可将该对象加入IOC容器中；
	3.我们之前学习过自动配置的原理就知道SpringBoot在启动过程中会加载所有jar包下Mate-info
	  下的spring.factors文件下以EnableAutoconfiguration为key的配置类全限定类名，然后将他们加载到BeanDefinitionMap中。我们可以借助
	  这个原理在我们本地项目中的resources下创建/META-INF/spring.factories，将我们创建的配置类添加到我们创建的spring.factors文件中；
	4.然后在需要使用到该配置的工程中引入即可；

### 热插拔技术
	1.热插拔技术可以说就是某种配置是否需要使用的一种开关，像SpringBoot项目中很多地方用到的以@Enable开头的注解就是热插拔拔技术的一种体现；
	实现方法如下：
	1.创建一个标识类,如：SignObject.class，该类不需要做任何事情，只是在加载配置类供SpringBoot以此类为判断标准，是否需要加载该配置类；
	2.自定义@EnableConfigSign，在该注解上添加@Import(SignObject.class)，这样就会将这个标识类放进IOC容器当中，我们知道SpringBoot在将
	  EnableAutoConfiguration对应的配置类加载后会根据该配置类上的@ConditionalOnClass(SignObject.class)的条件注解去判断是否加载该配置；
	  如该条件不成立则不实例化该配置类，从而达到配置开关的效果；
	  
### 内嵌Tomcat原理
	配置原理：
		1.Spring Boot默认支持Tomcat，Jetty，和Undertow作为底层容器。而Spring Boot默认使用Tomcat，
		一旦引入spring-boot-starter-web模块，就默认使用Tomcat容器。核心就是在该依赖的starter中引入了tomcat和SpringMvc；
		2.Tomcat的配置类也是在自动配置阶段完成的，在MATE-INFO下的自动配置列表中有一个ServletWebServerFactoryAutoConfiguration配置类；
		  进入该类，里面也通过@Import注解将EmbeddedTomcat、EmbeddedJetty、EmbeddedUndertow等嵌入式容器类加载进来了，springboot默认是启动嵌入式tomcat容器，
		  如果要改变启动jetty或者undertow容器，需在pom文件中将toncat依赖排除，然后引入需要替换的依赖，如：jetty；
		3.我们进入EmbeddedTomcat该对象，我们可以看到其通过创建了一个TomcatServletWebServerFactory工厂对象；我们进入该对象有一个getWebServer方法，沿着该方法的
		  调用链会找到initialize()方法，然后在改方法中执行了this.tomcat.start()启动Tomcat；
	如何调用配置（getWebServer）解析:
		1.其调用的入口则就是在启动类的run方法下的refreshContext方法里面，该方法最终会调用到AbstractApplicationContext类的refresh()方法；
		2.refresh方法中会调用了onrefresh方法，该方法是一个模板方法，被ServletWebServerApplicationContext对象实现，然后调用到该对象的createWebServer()；
		3.在该方法中就会调用上述中提到的TomcatServletWebServerFactory的Tomcat工厂，然后通过该工厂的getWebServer方法获取Tomcat对象并启动；
	小结：
		springboot的内部通过 new Tomcat() 的方式启动了一个内置Tomcat。但是这里还有一个问题，
		这里只是启动了tomcat，但是我们的springmvc是如何加载的呢？
		
### 自动配置SpringMVC（将servlet添加到Servlet容器中）
	我们之前使用SpringMvc框架是在web.xml中配置DispatchServlet的，现在没有web.xml这个配置文件了，我们是如何配置DispatchServlet的？
	其实Servlet3.0规范中规定，要添加一个Servlet，除了采用xml配置的方式，还有一种通过代码的方式，伪代码：servletContext.addServlet(name, this.servlet);
-	这个其实也是基于SpringBoot自动装配完成的，在mate-info文件下的spring.factors文件中配置了DispatcherServletAutoConfiguration这个配置类，而这个配置类
	主要是做了DispatcherServlet（配置Servlet）和DispatcherServletRegistry（将Servlet注册到Servlet容器中）这两件事，而这个配置类会在ServletWebServerFactoryAutoconfiguration
	配置之后加载，也就是web容器加载后执行；
-	小结：
		springboot mvc的自动配置类是DispatcherServletAutoConfigration，主要做了两件事：
		1）配置DispatcherServlet
		2）配置DispatcherServlet的注册Bean(DispatcherServletRegistrationBean)
-	接下来看下如何注册DispatcherServlet到ServletContext？
	org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#createWebServer
	它是在以上方法中调用了Servlet容器启动后在执行添加Servlet到Servlet容器中的；具体如下：
	注：首先onStartup这个方法是属于ServletContextInitializer这个接口的，而这个接口主要就是注册Servlet。下面的对象实现了这个接口；
		DispatcherServletRegistrationBean这个对象实现是没有onStartup这个方法，那就要调用它的父类RegistrationBean下的onStartup():
-	代码调用流程如下：
	org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#selfInitialize 注册入口方法
		org.springframework.boot.web.servlet.RegistrationBean#onStartup
			org.springframework.boot.web.servlet.DynamicRegistrationBean#register
				org.springframework.boot.web.servlet.ServletRegistrationBean#addRegistration 最后执行：servletContext.addServlet(name, this.servlet);完成注册
-	小结：
		SpringBoot自动装配SpringMvc其实就是往ServletContext中加入了一个 Dispatcherservlet 。
		Servlet3.0规范中有这个说明，除了可以动态加Servlet,还可以动态加Listener，Filter	
		
### 数据源自动配置
	1.引入对应数据库依赖后SpringBoot自动配置在加载自动配置类时会读取MATE-INFO文件下的spring.factors文件下DataSourceAutoconfiguration配置类并加载；
-	@Configuration(proxyBeanMethods = false)	配置类注解
-	@Conditional(PooledDataSourceCondition.class) 	根据判断条件，实例化这个类，指定了配置文件中，必须有type这个属性，另外springboot 默认支持 type 类型设置的数据源；
-	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
-	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class, DataSourceConfiguration.Dbcp2.class,DataSourceConfiguration.Generic.class,
		DataSourceJmxConfiguration.class })
-	protected static class PooledDataSourceConfiguration {}
-	PooledDataSourceConfiguration配置类是我们需要重点关注的类，并查看通过@Import注解导入的DataSourceAutoConfiguration四种数据库连接池配置。从而得出结论：
	1.DataSourceConfiguration.Hikari.class  //2.0 之后默认默认使用 hikari 连接池,
	2.DataSourceConfiguration.Tomcat.class, //2.0 之后默认不是使用 tomcat 连接池,或者使用tomcat 容器，如果导入tomcat jdbc连接池 则使用此连接池，
	  在使用tomcat容器时候或者导入此包时候。
	3.DataSourceConfiguration.Dbcp2.class	//（Dbcp2 连接池）。
	4.DataSourceConfiguration.Generic.class // 自定义连接池 接口 spring.datasource.type 配置）
-	并证实如下结果：
		1.org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration#createDataSource 使用DataSourceBuilder 建造数据源，利用反射创建type数据源，然后绑定相关属性；
		2.org.springframework.boot.jdbc.DataSourceBuilder#DATA_SOURCE_TYPE_NAMES（连接池类型数组）  org.springframework.boot.jdbc.DataSourceBuilder#getType（获取类型方法）
		  取出来的第一个值就是com.zaxxer.hikari.HikariDataSource，那么证实在没有指定Type的情况下，默认类型为com.zaxxer.hikari.HikariDataSource
		3.可以在全局配置文件中配置spring.datasource.type去设置数据库连接池类型；
		4.不同连接池的DataSource对象的properties属性与SpringBoot的默认的属性不能完全匹配，需要自定义配置；

### Mybatis自动配置解析
-	springboot项目最核心的就是自动加载配置，该功能则依赖的是一个注解@SpringBootApplication中的@EnableAutoConfiguration
-	EnableAutoConfiguration主要是通过AutoConfigurationImportSelector类来加载以mybatis为例，通过反射加载spring.factories
	中指定的java类，也就是加载MybatisAutoConfiguration类（该类有Configuration注解，属于配置类）
-	MybatisAutoConfiguration配置类中有两件比较重要的事，一个是创建sqlSessionFactory对象。一个是sqlSessionTemplate创建出来并加入IOC容器中，
	接下类分解解析下这两个配置类；
-	org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration#sqlSessionFactory（SqlSessionFactory的创建）
		1.创建Configuration对象。
		2.获取SqlSessionFactoryBean的getObject()中的对象注入Spring容器，也就是SqlSessionFactory对象
			org.mybatis.spring.SqlSessionFactoryBean#buildSqlSessionFactory 
				org.apache.ibatis.builder.xml.XMLMapperBuilder#parse （XML配置解析）这个方法已经是mybaits的源码，初始化流程
				org.apache.ibatis.session.SqlSessionFactoryBuilder#build 这个方法已经是mybaits的源码，初始化流程；
-	org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration#sqlSessionTemplate （往Spring容器中注入SqlSessionTemplate对象）	
		1.SelSessionTemplate，作用是与mapperProoxy代理类有关
		2.该Sqlsession的实现是线程安全的（可被多个线程共享），像我们平时调用的是DefulSqlSession(线程级，非线程安全)
- 	现在已经得到了SqlSessionFactory了，接下来就是如何扫描到相关的Mapper接口（@MapperScan(basePackages = “com.mybatis.mapper”)）
	1.通过@Import的方式会扫描到MapperScannerRegistrar类。MapperScannerRegistrar实现了ImportBeanDefinitionRegistrar接口，那么在spring实例化之前
	  就会调用到registerBeanDefinitions方法；
		org.mybatis.spring.annotation.MapperScannerRegistrar#registerBeanDefinitions
			org.mybatis.spring.mapper.ClassPathMapperScanner#doScan 
				org.mybatis.spring.mapper.ClassPathMapperScanner#processBeanDefinitions 将扫描到的Dao对象转类型为MapperScannerConfigurer然后注册到spring容器中；
-	小结：
		@MapperScan这个定义，扫描指定包下的mapper接口，然后设置每个mapper接口的beanClass属性为MapperFactoryBean类型并加入到spring的bean容器中。			
		MapperFactoryBean实现了FactoryBean接口，所以当spring从待实例化的bean容器中遍历到这个bean并开始执行实例化时返回的对象实际上是getObject方法中返回的对象。
		最后看一下MapperFactoryBean的getObject方法，实际上返回的就是mybatis中通过getMapper拿到的对象，这个就是mybatis通过动态代理生成的mapper接口实现类。
		
###  SpringBoot缓存深入		
-	JSR是Java Specification Requests 的缩写 ，Java规范请求，故名思议提交Java规范，JSR-107呢就是关于如何使用缓存的规范，
	是java提供的一个接口规范，类似于JDBC规范，没有具体的实现，具体的实现就是reids等这些缓存。
-	JSR107核心接口
	1.Java Caching（JSR-107）定义了5个核心接口，分别是CachingProvider、CacheManager、Cache、Entry和Expiry。
	2.CachingProvider（缓存提供者）：创建、配置、获取、管理和控制多个CacheManager。
	3.CacheManager（缓存管理器）：创建、配置、获取、管理和控制多个唯一命名的Cache，Cache存在于CacheManager的上下文中。一个CacheManager仅对应一个CachingProvider。
	  Cache（缓存）：是由CacheManager管理的，CacheManager管理Cache的生命周期，Cache存在于CacheManager的上下文中，是一个类似map的数据结构，并临时存储以key为
	4.索引的值。一个Cache仅被一个CacheManager所拥有；
	5.Entry（缓存键值对）：是一个存储在Cache中的key-value对
	6.Expiry（缓存时效）：每一个存储在Cache中的条目都有一个定义的有效期。一旦超过这个时间，条目就自动过期，过期后，条目将不可以访问、更新和删除操作。
      缓存有效期可以通过ExpiryPolicy设置。
-	Spring的缓存抽象
	Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用Java Caching（JSR-107）
	注解简化我们进行缓存开发。Spring Cache 只负责维护抽象层，具体的实现由自己的技术选型来决定。将缓存处理和缓存技术解除耦合。
	每次调用需要缓存功能的方法时，Spring会检查指定参数的指定的目标方法是否已经被调用过，如果有就直接从缓存中获取方法调用后的结果，
	如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
-	使用Spring缓存抽象时我们需要关注以下两点：
   	1.确定那些方法需要被缓存
   	2.缓存策略
-	重要概念&缓存注解：
	1.Cache：缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等。
	2.CacheManager：缓存管理器，管理各种缓存(Cache)组件。
	3.@Cacheable：主要针对方法配置，能够根据方法的请求参数对其结果进行缓存。
	4.@CacheEvict：清空缓存
	5.@CachePut：保证方法被调用，又希望结果被缓存。
	6.@EnableCaching：开启基于注解的缓存（作用于主类上）。
	7.keyGenerator：缓存数据时key生成策略。
	8.serialize：缓存数据时value序列化策略。
-	说明：
	1 @Cacheable标注在方法上，表示该方法的结果需要被缓存起来，缓存的键由keyGenerator的
	策略决定，缓存的值的形式则由serialize序列化策略决定(序列化还是json格式)；标注上该注解之
	后，在缓存时效内再次调用该方法时将不会调用方法本身而是直接从缓存获取结果
	2.@CachePut也标注在方法上，和@Cacheable相似也会将方法的返回值缓存起来，不同的是标
	注@CachePut的方法每次都会被调用，而且每次都会将结果缓存起来，适用于对象的更新
		
### 缓存自动配置原理源码剖析
	1.当AutoConfigurationImportSelector组件导入自动配置类的时候会添加一个CacheAutoConfiguration的缓存自动配置类，该类下有
	  有一个静态内部类 CacheConfigurationImportSelector 他有一个 selectImport 方法是用来给容器中添加一些缓存要用的组件;
	2.在添加的组件列表中只有SimpleCacheConfiguration缓存组件会被调用，说明这个缓存注解是默认的缓存管理器，而这个对象中创建
	  ConcurrentMapCacheManager这个cachaManage缓存管理器，这个缓存管理器本质上使用的ConcurrentMap数据接口缓存数据。
	3.@Cacheable注解运行流程：
		1.方法运行之前，先去查询Cache(缓存组件)，按照cacheNames指定的名字获取(CacheManager先获取相应的缓存，第一次获取缓存如果没有Cache组件会自动创建)。
		2.去Cache中查找缓存的内容，key默认就是方法的参数，key默认是使用keyGenerator生成的，默认使用的是SimpleKeyGenerator；
		3.没有查到缓存就调用目标方法。
		4.将目标方法返回的结果放进缓存中。
		总结：@Cacheable标注的方法在执行之前会先检查缓存中有没有这个数据，默认按照参数的值为key查询缓存，如果没有就运行方法并将结果放入缓存，再来调用时直接使用缓存中的数据。
		
###  基于Redis的缓存实现
-	SpringBoot默认开启的缓存管理器是ConcurrentMapCacheManager，创建缓存组件是ConcurrentMapCache，将缓存数据保存在一个个的ConcurrentHashMap<Object, Object>中。
	开发时我们可以使用缓存中间件：redis、memcache、ehcache等，这些缓存中间件的启用很简单——只要向容器中加入相关的bean就会启用，可以启用多个缓存中间件
-	引入redis的starter之后，会在容器中加入redis相关的一些bean，其中有两个跟操作redis相关的：RedisTemplate和StringRedisTemplate(用来操作字符串：key和value都是字符串)，
	template中封装了操作各种数据类型的操作(stringRredisTemplate.opsForValue()、stringRredisTemplate.opsForList()等)	
	
	
	
	
	
	
	
	
	
	
	

	