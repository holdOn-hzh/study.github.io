# Spring框架

### Spring的简介： 
是分层全栈式轻量级（启动时加载资源较小，只需要一个基本环境，无需引用第三方）开源框架，以IOC和AOP为核心思想，
提供了展现层Spring-mvc框架集成和业务层事务支持等应用，还能集成第三方开源框架；

### Spring的优势：
- 方便解耦，简易开发：使用IOC容器管理管理bean以及对象依赖关系；解决了硬编码所带来的过度耦合；
- Aop编程支持：使用Aop面向且面编程，为业务提供横向扩展；
- 声明式事务的支持（@Transactional）通过声明式方式灵活的进行事务的管理，提高开发效率和质量。
- 方便程序的测试：可以用非容器依赖的编程方式进行几乎所有的测试工作，测试不再是昂贵的操作，而是随手可做的事情。
- 方便集成各种优秀框架：Spring可以降低各种框架的使用难度，提供了对各种优秀框架直接支持；
- Spring源码是经典的 Java 学习范例

### Spring 的核心结构：
- spring框架使用模块化思想，当在开发中需要哪个模块引用即可，无需集成全部spring模块
- Spring核心容器（Core Container） 容器是Spring框架最核心的部分，它管理着Spring应用中bean的创建、配置和管理。
  在该模块中，包括了Spring bean创建，它为Spring提供了DI的功能；
- 面向切面编程（AOP）/Aspects Spring对⾯向切⾯编程提供了丰富的支持。这个模块是Spring应用系统中开发切⾯的基础，与DI一样，AOP可以帮助应用对象解耦。
- 数据访问与集成（Data Access/Integration）；
- Web 该模块提供了SpringMVC框架给Web应用，还提供了多种构建和其它应用交互的远程调用方案；
- Test 为了使得开发者能够很方便的进行测试，Spring提供了测试模块解决Spring应用的测试；

### IOC（控制反转）的容器的理解：
- IOC其中有两个关键点：控制（对象的创建与管理） 反转（由自己创建对象改为有容器创建）
- IOC容器所解决的问题：解决由于硬编码问题带来的对象深度耦合，主要针对：对象创建和对象依赖关系的管理，
  我们在开发的过程中都是基于接口编程，而如果接口产生了多个实现，那么就需要手动修改对象实例的创建，
  有了IOC容器，我们就不需要担心这些事；
  
### DI（依赖注入）的理解：
- DI和IOC所指的都是同一件事，只不过角度不同，IOC是将在对象的角度（对象里声明的对象由容器注入），
  而DI所站的是容器的角度，对象里声明的对象需要容器注入对应的实例；
  
### AOP思想理解：
- AOP思想的总纲是面向切面编程（需要了解什么是垂直纵向的继承体系，和切面 横向体系）
- AOP解决的问题是当OOP（继承，多态，封装）面向对象编程所解决不了的代码重复问题，
  OOP面向对象属于垂直纵向继承体系，一般来说足以解决大多数代码重复问题，但是在垂直
  继承体系的顶级父类下的子流程（实现方法）中存在的代码复用问题无法解决；
- 我们把在垂直体系中添加的（增强代码）称之为横切逻辑代码
- AOP的使用场景非常有限，如：事务控制，权限校验，日志记录

### AOP如何解决问题，解决什么问题？
- AOP利用横向抽取机制，将业务代码和横切逻辑代码进行分开，解决业务代码与横切逻辑混乱的问题；
- 在不影响原有的业务逻辑的基础上添加横切逻辑，简单来说就是解决业务代码和横切代码的耦合关系；

### 切面编程的“切”和“面”
- [切] 指的是垂直纵向的继承体系当中在不影响原有业务逻辑的基础上添加横切代码；
- [面] 指的是往往“切”的不会是单独的一个方法，一个方法代表一个点，而又多个点组成了面，所以叫面向切面编程；

### IOC和AOP自定义实现思路解析：
- IOC主要实现对象创建及依赖关系控制反转，而创建对象除了new关键字外就是利用反射实现，但是反射实现需要有“全限定类型”的前提。
  我们可以通过xml配置的形式将需要创建的对象及依赖关系配置，再通过BeanFactory工厂的通过反射创建对象并解析对象中声明的对象属性
  通过setting方法赋值，将创建好的对象存储在map中。
- 事务控制则需要添加横切逻辑代码，在每一个顶级父类下的多个子实现（实现方法）添加事务控制，为提高代码复用性。可以使用的思路是：
  1.使当前线程使用的connation连接都使用同一个，也就需要将线程与连接绑定在一起（ThreadLocal）。
  2.在业务层中实现事务控制，可以将事务控制在类的级别也可以将事务控制在方法的级别（通过注解标注）
  
### 代理模式分析
- 代理：我们的代码就是用来描述生活的，我们可以打个比方，代理就相当于中介，把本来我们需要自己干的事委托了出去，而自己只关注结果就行；
- 静态代理：是我们只是针对某一个业务的扩展而手工添加的代理处理类，它不能对多个业务处理。
- 动态代理：是在运行期我们可以动态的为一些需要扩展的对象进行代理增强，动态代理可以使用（JDK动态代理/CGLIB动态代理）。
  这两种动态代理的实现底层和实现方式有所不同，具体如下：
  1.JDK动态代理是以实现代理对象的子类，所以在使用的JDK动态代理时代理对象必须实现接口；
  2.CGLIB是一种操作字节码框架，所以它实现代理的方式就是修改字节码文件，使用时代理对象不需要实现接口，在性能上也会有一些优势，但是需要引入依赖；
  
### 实现spring Ioc容器的基本思路：
通过前面的实践我们都知道可以通过配置和反射机制实现容器管理，但是在spring中有多种配置形式：
1.全XML配置bean定义信息，这种配置形式在javaSE中使用（ClassPathXmlApplicationContext/FileSystemXmlApplicationContext对象加载）
	javaWeb应用中使用（ContextLoaderListen监听器实现）
2.XML+注解配置bean定义信息，与全xml配置形式相同
3.注解形式配置bean定义信息，javaSE中使用（AnnocationConfigApplicationContext加载）
    在javaWeb中还是使用ContextLoaderListen监听器实现，不过此时处理的就是注解配置处理类；
	
### BeanFactory和ApplicationContext的区别
- BeanFactory是IOC容器的顶级接口，具备了容器的所有的基础方法；
- ApplicationContext是BeanFactory的一个子接口（高级接口），具备了比BeanFactory更全面的功能；
  比如：国际化支持和资源访问等等
  
### 实例化Bean的3种方式（学习这一块时需要区分实例化对象和DI注入）
- 无参构造：这是最常用的一种方法，在通过反射实例化bean的过程中调用类的无参构造方法对属性进行赋值；
- 静态方法：我们自己创建一个bean对象，实现静态方法并返回需要管理和创建的对象，在application.xml中
  创建bean标签，指定id和静态工厂对象，及对应的静态方法（factory-method）；
- 实例化方法：这种是通过实例化对象的实例化方法进行对象管理；
- 一般我们常用的是使用无参构造赋值，静态/实例方法赋值一般用于xml难以表述的对象创建；

### bean的作用范围
- bean的作用范围有6种，分别是：singleton(单例)，prototype(原型)，request(请求)，session(会话)，application(应用)，websocket(通讯)
- 单例指的是IOC容器当中只有一个对象。原型是每次执行getting方法时都会创建一个全新的对象，request是每一次请求创建新对象，session是
  每一次会话创建新对象；
- 大多数情况下我们都是使用singleton/prototype这两种scope，其他的一般应用于WEB应用；

### 对象的生命周期xml配置
init-method：在对象创建前执行 distory-method在对象销毁之前执行；

### DI注入 (xml配置)
- 注入的方式分为：set方法注入<property>和构造方法注入<constructor>
- 注入的类型有bean类型，基本类型，复杂类型（Map，List，Set）；

### Spring高级特性
- lazy-init（延迟加载）指的是在spring容器初始化阶段不实例化该对象，在调用容器的getBean()方法的时候再进行初始化；
  验证方法：在没有调用getBean方法前在ApplicationContext下的BeanFactory下的singleObjects(单例池)下可以查看对象是否存在；
- BeanFactory和FactoryBean：BeanFactory是IOC容器的顶级接口，里面包含的容器的基本行为。FactoryBean我们可以利用它自定义对象
  的创建过程，一般用于复杂对象的创建，FactoryBean调用的时候返回的并不是对象本身，而是FactoryBean中getBean方法中返回的对象；
- 后置处理器：BeanPostProcessor（Bean对象生命周期增强）和BeanFactoryPostProcessor（Bean工厂声明周期增强）

### Spring的Bean工厂创建
- 在对Spring的bean对象进行生命周期管理之前我们需要先创建BeanFactory对象，这个对象要根据配置形式解析Bean，如果是xml配置的话就需要
  将配置文件加载到内存中，如果是注解形式测需要扫描配置类和扫描需要被管理的对象；
- 在这个阶段最重要的事情就是将每一个bean对象转化成BeanDefinition对象，BeanDefinition对象中描述了当前对象定义信息和创建过程描述；
  如：bean的作用范围（scope），构造函数，是否懒加载（lazy-init）,对象依赖，是否单例等信息；

### Spring的bean生命周期流程（BeanFactory声明周期要在Bean对象生命周期之前）
1. 实例化bean（创建该对象） 
2. 设置属性（根据BeanDefinition中描述的依赖关系对bean的属性进行赋值）
3. BeanNameAware(可通过该接口setBeanName方法为bean设置名称)
4. BeanFactoryAware（Spring为实现该接口的对象传入了BeanFactory对象，可以根据此对象对BeanFactory进行修改）
5. ApplicationContextAware（Spring为实现该接口的对象传入了ApplicationContext高级接口，可以对容器进行修改）
6. BeanPostProcessor（该接口提供了两个方法before/after，可以在初始化bean的前后进行处理，在此处调用该接口的before前置处理）
7. InitializingBean（设置属性）
8. 调用对象的init-method方法
9. 调用BeanPostProcessor接口的after方法
   在这个阶段会判断bean是否是单例还是多例的，单例的话会放进BeanFactory的singleObjects(单例池)，多例的则返回给调用，Spring后期不做干涉；
10. 执行destory(对象销毁)

### BeanPostProcessor 和 BeanFactoryPostProcessor 
- 针对的是不同的对象，前者是为Bean生命周期进行后置处理，后者是为BeanFactory工厂进行后置处理；

### IOC容器结构分析
BeanFactory是容器的顶级接口，定义了容器的基本行为，但是spring不会把所有的功能都放在一个接口，而是把容器的大功能全部打散，这样子实现就能根据自己的
需要去实现对应接口，比如：ListableBeanFactory,MessageSourse,ResourceLoard,ApplicationContext;

### IOC容器流程分析
Spring容器主流程
	1.BeanFactory生命周期
		1.创建beanFactory实例
		2.加载BeanDefinition对象（描述bean的定义信息和创建过程）
		3.beanDefinition注册到beanDefinitionRegistry(map)中
			1.进行资源读取，将bean信息转成Resouce对象
			2.初始化NameSpaceHandlerResouver对象进行xml配置元素前缀处理；
			3.通过BeanDefinitionHolder包装BeanDefinition对象进行元素加载
			4.将创建的BeanDefinition注册到BeanDefinitionRegistry中
			5.将BeanDefinition放进BeanDefitionMap中
		4.BeanFactoryPostPocessor处理
		5.BeanDefinitionPostPocessor执行
	2.FactoryBean对象生命周期	 ？
	
### IOC容器初始化过程个人理解 （所有的执行流程都是在Refresh方法中执行的）
	1.配置信息预处理
	2.初始化BeanFactory工厂（refreshBeanfactory方法初始化BeanFactory和加载BeanDefinition对象，解析对应的配置，如使用nameSpaceHandlerResolver解决xml配置的前缀）
	3.准备BeanFactoryPostPoccessor接口的执行，检查实现了beanFactoryPostPocessor接口的对象
	4.执行BeanFactoryPostPocessor的后置处理方法
	5.注册BeanPostProcessor后置处理器
	6.为上下文初始化国际化支持
	7.初始化上下文事件监听器
	8.在特定上下文子类中初始化其他特殊bean
	9.检查侦听器bean并注册它们
	10.初始化单例并且非延迟加载的对象
	11.发布对应的事件
	
### spring循环依赖解决（基于java对象的引用实现）
- 什么是循环依赖？
  答：指的是现在有两个对象A和B，A的属性中有B，B的属性中有A；两个对象互相引用；
- 循环依赖不能解决的情况有哪些？
  答：第一种情况就是构造器注入，1.是因为构造器是初始化对象时才调用，spring在解决循环依赖过程中不会去初始化2次，而是构造器注入必须有参数，但是在一开始的时候并没有依赖可以注入；
	  第二种就是多例的对象无法注入，因为spring将多例的对象创建之后是不对其进行管理的；
- 循环依赖如何解决？
  答：利用3级缓存解决，singletonObjects（单例池，完整对象），二/三级缓存（非完整对象）
      在创建A对象时发现属性对象没有创建，则将提前将自己实例化暴露在3级缓存中，然后再去调用B依赖对象，这时B依赖对象会从3级缓存逐一获取到属性对象将其注入，然后B依赖对象将自己放入
	  2级缓存，此时B依赖对象已经是完整对象，可以直接放入单例池中，而3级缓存中的A对象所依赖的B对象已经实例化完毕，则将3级缓存中的A对象放入单例池中；
- 为什么需要3个缓存区（明明2个就可以解决）
  答：答案在这个方法中org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#getEarlyBeanReference
	  再将3级缓存中的对象放到2级缓存的时候会执行getEarlyBeanReference()这个方法会将去执行BeanPostProcessor后置处理器进行扩展
	  
### 循环依赖过程解析
- 首先所有的bean都会走3级和1级缓存，如果循环依赖中依赖的bean是个代理对象，则会去产生创建代理对象流程，将它放到二级缓存中；
- 1.创建bean对象时首先将自己提前暴露在3级缓存中
- 2.从3级缓存依次寻找依赖的属性对象，找到后赋值给当前对象，当前对象赋值完毕是完整对象时直接放入1级缓存，删除在3级缓存中的对象；
- 3.所依赖的对象是代理对象，则触发扩展机制通过其中的advise后置处理器创建代理对象，将代理对象注入到当前对象，然后将代理对象放进2级缓存，删除3级缓存对象；
- 总结：bean一般都经历3级和2级缓存，只有产生循环依赖和依赖对象是代理对象时才会使用二级缓存；
       不使用2级缓存也可以，直接将对象放入1级缓存，但是缺违反了对象单一职责；
	  
### AOP术语描述（锁定具体的切入位置）
- 切入点:需要添加横切逻辑代码的方法
- 连接点：具体方法中的位置，如：方法执行前/后，异常等等
- advice（增强）：指的是横切逻辑代码（前置通知，后置通知，正常通知，异常通知，环绕通知）
- targe对象：需要被增强的 被代理对象
- proxy代理：我们是通过代理对象实现横切逻辑代码织入的
- 织入：将横切逻辑代码在与业务代码无耦合的情况下执行
- 切面：在垂直纵向的继承体系中添加横切逻辑代码，而切入点不会只有一个，所以由多个切入点组成了一个面，所以叫切面编程；
	
### Spring事务支持
- 编程式事务：是指把事务控制代码和业务代码耦合在一起的一种事务控制手段
- 声明式事务：是通过注解/xml进行配置，通过AOP与业务代码进行解耦达到事务控制
- 事务的四大特性：
	原子性：站在业务的角度，指的是一组操作，要么全部成功，要么全部失败；
	一致性：站在数据的角度，对数据操作的结果要保持一致，比如转账操作，他们的操作的结果后的和肯定是不变的；
	隔离性：也就是事务和事务之间互不影响；
	持久性：是将每次的操作的记录都存储在磁盘中，不会将结果丢失；
- 隔离级别带来的问题：
	脏读：读取了事务未提交的数据。
	不可重复读：两次读取的数据不一样，之前读取的数据已经不能再读了，主要是update带来的问题。
	幻读：两次读取的数据记录条数不一样，主要是insert或delete带来的问题。
- 4种隔离级别：
    串行化：隔离级别最高的，加锁力度最强，性能最差；
	不可重复读：可解决脏读，不可重复读的情况，会出现幻读的情况，对update加锁;
	读已提交：解决了脏读的情况，但是会出现幻读或者不可重复读的；
	读未提交：没有任何解决作用，级别最低，什么情况都有可能出现；
	
### AOP实现原理描述
	AOP是Spring通过动态代理实现的，在实现接口的情况下默认使用JDK动态代理实现，没有实现接口的话使用CGLIB实现动态代理；
	实例化过程是：在容器启动时在refresh方法种调用初始化bean方法，在容器查无当前对象的情况下调用getBean()方法，然后在创建
	对象后调用后置处理，根据对应的动态代理技术要求通过BeanPostProcessor方法的after生成代理对象，在createProxy方法中将
	Aop增强和通用拦截器合并，交给代理对象，然后调用getProxy方法获取代理对象；
	
### @EnableTransactionManagement声明式事务实现原理
	在EnableTransactionManagement注解中使用@Import注解引入了两个组件，事务属性解析器和事务拦截，基于BeanPostProcessor后置处理
	器进行事务增强，可以说声明式事务就是AOP思想的一种实现。
	事务属性解析器是用来解析@Transactional注解中的配置信息；事务拦截器器是在产生代理对象之前于aop进行合并，最终产生代理对象；
	
	

疑问:1.课程中老师演示了beanFactory，beanDefinition，bean创建的源码执行过程，这些该怎么总结？怎么表述？
     2.面试中经常被问spring管理的bean 是否线程安全，我的疑问是线程安全和spring有关系吗？我的理解是spring框架
	   只是搭了台子，安不安全不是由开发者实现吗？毕竟IOC管理对象是没有竞争关系的，只有bean对象的行为才能产生线程安全
	 3.aop里面的有5种连接点，前置通知，后置通知，正常通知，异常通知，环绕通知；其中正常通知和后置通知有什么区别？它们能同时使用吗？

  
  
	


  
  














  
  
	