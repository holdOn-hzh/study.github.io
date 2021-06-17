## springMVC

### springMVC简介
SpringMVC是基于MVC设计模型应用于表现层的轻量级web框架。是对于原生servlet的封装，原先我们有多少个业务就需要通过实现接口的方法实现多少个servlet，
而现在SpringMVC对servlet做了封装，只需要由dispatcherServlet前端控制器和客户端进行交互，由dispatcherServlet前端控制器 根据规则转发请求到对应的控制器，
然后返回结果由DispatcherServlet响应给客户端；

### 执行基本流程
定义dispatcherServletHandler前端控制器，mapper-url拦截路径配置；
xml配置扫描包，视图解析器，处理器映射器；
dispatcherServlet与前端进行交互，接收客户端请求，然后调用mapperHandler查询对应的处理器和拦截处理器（如果有）;
mapperHandler根据url返回给disPatcherServlet,由于前端控制器去调用对应的handlerAdapter适配器（由于handler的形式有多种，比如注解定义，接口实现，原生servlet等需要适配）
执行handler,handler返回对应的modleAndView，包含了视图和数据；SpringMVC中的各大组件没有耦合关系，全部和dispatcherServlet交互；

### SpringMVC九大组件
- mapperHandler（相当于handler注册表）前端控制器会调用当前组件，当前组件需要根据Url查询对应的handler和intecepter;
- handlerAdapter（处理器适配器）由于handler的形式有多种，比如注解定义，接口实现，原生servlet等，所以需要进行适配，在调用handler执行；
- handlerexceptionResolver(处理器异常处理)当handler发生异常时需要处理的事，比如异常封装，异常情况下的视图封装；
- ViewResolver(视图解析器)根据定义好的前缀和后缀进行拼接，找到对应的视图。主要是为了解决视图名称硬编码的问题；
- RequestToViewNameTranslator(请求转视图名)在当前请求里面没有传视图名称，当前组件会从请求中找出视图名称，比如请求路径；
- LocaleResolver（国际化）解析当前区域，比如中国的locale是zh-CH（表示一个区域）；
- ThemeResolver（主题解析器）ThemeResolver 组件是用来解析主题的。主题是样式、图片及它们所形成的显示效果的集合。
- MultipartResolver（用于上传文件）通过将普通的请求包装成 MultipartHttpServletRequest 来实现。可以通过getFile获取文件
- FlashMapManager FlashMap 用于重定向时的参数传递；


### SpringMVC的url-pattern拦截规则
1.以通配符的形式如：*.do
2./符，这种方式会拦截所有请求，不会拦截jsp请求，但是会拦截静态资源请求，如：html,js;
3./*符号，这种方式会拦截所有请求，包括了jsp，静态资源
	

### /符号拦截规则问题解析
1.为什么会拦截静态资源？：是因为Tomcat容器中也有一个web.xml,我们的项目中也有web.xml,相当于我们这个子web.xml覆写了父web.xml的规则，
  所以走的我们当前项目中实现的servlet规则；
2.为什么不会拦截jsp请求？：是因为Tomcat容器的实现了jspServlethandler处理jsp请求，我们并没有覆盖这个配置；	
3.解决方案有两种
	1.配置<mvc:default-servlet-handler>标签，会在SpringMVC上下文中定义一个DefaultServletHttpRequestHandler对象，相当于拦截器会去
	  检查是否是静态文件请求，是的话则由Toncat的servlet处理器处理；但是有位置限制，静态文件必须在webapp根目录下；
	2.<mvc:resources location="classpath:/"  mapping="/resources/**"/>配置springMVC自己的静态资源处理，location是文件位置，mapping
	  是对应的url路径，这种方式是没有文件位置限制的
	 
### SpringMVC接口处理方式描述
- Modul，ModulMap，Map定义入参，这三种方式都是实现了Map接口的，本质上就是BindMap，SpringMVC的封装模型对象；
- Servlet原生对象的获取如：HttpServletRequest,HttpServletSession，只要要在方法入参上声明即可获取；
- Servlet方法参数传递：基本类型传参，POJO对象传参，日期类型传参，可基于@RequestParme注解绑定映射关系；
- 自定义类型转换器，可以通过实现Converter接口完成类型处理，配置FormattingConversionServiceFactoryBean对象，
  将我们创建的类型转换器赋值给这个对象，然后注册到SpringMVC的处理映射器；

### REST风格标准（资源-状态-转换）
-	意思就是把客户端和Servlet的每一次请求都当成是对资源的一次操作，然后把操作细分为（GET，POST，PUT，DELETE）
	增删改查4种类型，这样子带给我们更加直观的感受，也更加的简洁。
	即使在URL相同的情况下SpringMVC也能根据你的请求类型找到对应的handler处理请求，可以通过@PathVariable注解获取
	在URL中的参数信息；
-	POST请求乱码问题，可以通过配置拦截器的形式将每一次POST请求的数据转成UTF-8；
-	SpringMVC的类型转换器，请求一般通过GET/POST传递，然后可以通过配置-method属性指定（PUT/DELETE）操作类型。
	这种方式也是需要配置拦截器的，可以拦截所有的POST请求，检查是否配置了_method属性，如果配置了则将POST请求
	类型转换成指定的REST风格请求方式；
	
### 过滤器 && 监听器 && 拦截器
- Servlet主要是处理request和response；
- 过滤器和监听器是属于JAVAEE中的应用，主要配置在web.xml,而拦截器属于表现层，配置在自己的配置文件中；
- 过滤器：主要应用于servlet之前，对request进行过滤，可以对所有请求包括静态文件和jsp请求；
- 监听器：是实现于ServletContextListener服务端组件，随着web应用的开启而初始化，随着web应用的关闭而销毁；
  会一直进行监控，它的作用一般有做一些初始化，监听一些web中的特定事件，比如:Request,Session的创建和销毁，在线人数统计；
- 监听器：监听器是属于框架本身的技术实现，主要是监听Handler，监听时机分别是：handler执行之前，handler执行之后返回视图之前，
  handler返回视图之后3个点；
  
### springMVC的拦截器使用和执行流程
- 一般实现过滤，监听，拦截这种扩展机制都是要实现某个接口然后进行注册，拦截器的实现也是这样；
- 首先实现HandlerInteceptor接口，实现三个时机进行拦截的方法，preHandler(handler执行前)，postHandler(handler执行后未返回视图)，afterHandler（视图渲染后）；
  preHandler的应用场景相对校对，一般用于权限校验；
- 实现接口后在配置文件中添加<mvc:inteceptors>配置<mvc:inteceptor>标签，添加拦截路径，排除路径，自定义实现的拦截器bean；
- 执行流程是preHandler，handlerAdaptor(handler执行)，postHandler，（render）视图渲染完毕，afterHandler；

### SpringMVC多个拦截器的执行流程
执行顺序是和配置顺序有关系的，而却每一个时机点的顺序也是不同的，preHandler时机点是根据配置顺序的正序执行，
剩下的执行时机点是根据配置顺序的倒序执行；

### MultipartFile文件上传，异常处理机制，转发，重定向
- 可以在接口中声明MultipartFile对象支持文件上传，传统的文件存储是将文件转成二进制存在数据库中，现在大部分是存在磁盘中，然后将路径存到数据库；
- 异常处理机制可以分为局部处理/全局处理，局部处理可以在当前的controller中添加个标记了@ExceptionHandler的方法，全局则可以声明一个全局异常处理类
  使用@ExceptionAdvice标记该类，然后可以在该类下声明不同的方法针对不同的异常，在方法上标记@ExceptionHandler声明；
- 转发对于定于来说是隐形的，只需要一次请求，虽然当前handler处理不了但是会去调用对应的handler,参数会丢失，可以通过url拼接传参，但是有数据量限制；
- 重定向则会产生两次请求，当前handler不做任何处理，只告知请求去调用哪一个handler,可以基于flashAttribute属性将参数绑定在session中；
  
  
### DispatchServlet源码刨析（主体框架）
- 继承关系：HttpServlet -> HttpServletBean -> FrameworkServlet -> DispatchServlet
- 由frameworkServlet实现doPost方法，在doPost方法中调用processRequest方法，processRequest方法调用了doService方法；
- doService具体实现由DispatchServlet实现，在doservice方法中做了上一次请求的一些信息处理，国际化处理，请求域信息处理，并调用doDispatch方法；
- doDispatch方法是springMvc中的核心方法，在此接受请求，handlerMapper匹配，调用handlerAdapter匹配handler,handler执行等等，于各大组件进行交互；

### doDispatch方法调用过程
- getHandler方法获取能处理当前handler的执行链；
- getHandlerAdapter方法调用，获取对应的handler适配器（根据handler获取不同的反射执行器）
- ha.handler适配器调用handler执行（返回ModleAndView的对象）
- 调用processDispatchResult完成视图跳转和渲染

### getHandler方法解析
	首先会去遍历MapperHandler的执行器链，其是个接口，有两种实现类，BeannameUrlMapperHandlerMapper（早期通过配置文件的形式）
	和RequtstMapperHandlerMapper（注解扫描的方式）；
	遍历的时候会根据request中的url去匹配对应的handler,并返回拦截器 intecepter
	
### getHandlerAdapter方法解析
	HandlerAdapter也是有多种实现的，有注解配置形式和实现Controller方法实现方式，在获取适配器的时候会根据handler中的对象是否实现Controller判断选择那种适配器
	
### SpringMvc9大组件的初始化
	SpringIOC容器在AbstractAunocationApplicationContext执行refrech方法中会调用onRefrech方法，该方法是个模板方法，被DispatchServlet实现，在该方法中调用initStartgain方法
	进行9大容器的初始化；
	（1）HandlerMapper：初始化策略，首先通过一个默认boole变量判断，为true的话通过BeanFactory中获取类型为HandlerMapper.class的handler对象。
	可以通过web.xml去配置这个boole变量将其改为false，则直接在context容器中通过“handlerMapper”名称取获取。在获取不到的情况下则会通过默认策略获取handler对象。
	springMvc会配置一个默认配置dispatchServlet.propetis文件，根据class类型获取默认的对象，比如HandlerMapper.class
	
### handler执行细节 && proccessDispatchResult（视图渲染过程）
	handler：进行参数解析，参数匹配，判断是否Servlet原生对象，通过封装好的HandlerMhthod对象中的handler信息通过反射机制执行handler获取对应的结果；
	proccessDispatchResult方法中根据传入的handler判断响应状态（转发，重定向），通过视图解析器解析视图名称，将逻辑地址转化为物理地址。创建view视图对象，
	然后将数据模型中的数据添加到request请求域中，这也是为什么jsp文件可以在请求域中获取数据的根本原因；
	
	
## SpringDataJPA（持久层操作框架）
	它是属于SpringData家族中的一部分，由实现JPA规范的持久层框架对数据库进行交互，比如hiberanate持久层框架；
	引入jar包和配置文件，创建与字段映射的实体类，再创建dao对象并继承实现了jpa规范的两个基本接口，通过调用方法与数据库交互；
	1.在JPA基本接口中已经定义好了基本的增删改查方法和复杂查询接口；
	2.可以引入Jpql的形式自定义方法，在方法上添加对应的增删改查注解添加对实体类的具体操作；
	3.以入原生sql，与Jpql的形式差不多，在注解内指定是原生sql类型，并编写对应的sql;
	4.方法命名规则方法，通过方法名指定对应的条件，比如 ByIdAndname;
	5.使用Specification对象支持动态sql,在方法中实现方法对sql进行拼接；





  
  
  





 
	
	
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  