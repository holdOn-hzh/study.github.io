### Mybatis
	MyBatis是一款优秀的基于ORM的半自动轻量级持久层框架，它支持定制化SQL、存储过程以及高级映
	射。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MyBatis可以使用简单的
	XML或注解来配置和映射原生类型、接口和Java的POJO （Plain Old Java Objects,普通老式Java对 象）
	为数据库中的记录。

## 自定义持久层框架
### JDBC问题总结：
1. 数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能；
2. Sql语句在代码中硬编码，代码不易维护，实际开发中sql变动需要改变java代码；
3. 使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，不支持动态sql的生成，修改sql还要修改代码，系统不易维护；
4. 对结果集解析存在硬编码(查询列名)，sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成pojo对象解析比较方便

### 问题解决的思路：
1. 使用数据库连接池初始化连接资源；
2. 将sql语句抽取到xml配置文件中；
3. 使用反射、内省等底层技术，自动将实体与表进行属性与字段的自动映射；

### 实现流程（自定义一个简单orm框架来理解Mybatis）
使用端提供核心配置文件：
	1.sqlMapConfig.xml : 存放数据源信息，
	2.引入mapper接口,Mapper.xml（sql语句的配置文件信息）
框架端：
	1.读取配置文件，读取完成以后以流的形式存在，我们不能将读取到的配置信息以流的形式存放在内存中，不好操作，可以创建javaBean来存储：
		（1）Configuration : 存放数据库基本信息、Map<唯一标识，Mapper> 唯一标识：namespace + "." + id
		（2）MappedStatement：sql语句、statement类型、输入参数java类型、输出参数java类型
	2.解析配置文件，创建sqlSessionFactoryBuilder类：
		（1）使用dom4j解析配置文件，将解析出来的内容封装到Configuration和MappedStatement中
		（2）创建SqlSessionFactory的实现类DefaultSqlSession	
	3.创建SqlSessionFactory：方法：openSession() : 获取sqlSession接口的实现类实例对象	
	4.创建sqlSession接口及实现类：主要封装crud方法。具体实现：封装JDBC完成对数据库表的查询操作
	5.使用代理模式来创建接口的代理对象来解决：
		（1）dao的实现类中存在重复的代码，整个操作的过程模板重复(创建sqlsession,调用sqlsession方法，关闭 sqlsession)
		（2）dao的实现类中存在硬编码，调用sqlsession的方法时，参数statement的id硬编码		
		
### Mybatis缓存
	1.缓存就是内存中的数据，常常来自对数据库查询结果的保存，使用缓存，我们可以避免频繁的与数据库进行交互，进而提高响应速度
	  Mybatis也提供了对缓存的支持，分为一级缓存和二级缓存。	
		（1）一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。
			 不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。
		（2）二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的
	2.一级缓存中如果查不到的话，就从数据库查，在queryFromDatabase中，会对localcache进行写入。 localcache对象的put方法最终交给Map进行存放；
	  具体方法：org.apache.ibatis.executor.BaseExecutor#queryFromDatabase
	3.二级缓存的原理和一级缓存原理一样，第一次查询，会将数据放入缓存中，然后第二次查询则会直接去缓存中取。但是一级缓存是基于sqlSession的，
	  而二级缓存是基于mapper文件的namespace的，也就是说多个sqlSession可以共享一个mapper中的二级缓存区域，并且如果两个mapper的namespace 相
	  同，即使是两个mapper,那么这两个mapper中执行sql查询到的数据也将存在相同的二级缓存区域中。	
	4.和一级缓存默认开启不一样，二级缓存需要我们手动开启：
		1.首先在全局配置文件sqlMapConfig.xml文件中加入如下代码:<setting name="cacheEnabled" value="true"/>
		2.其次在Mapper.xml文件中开启缓存：<cache></cache>
		
###  Mybatis插件
	1.在四大组件(Executor、StatementHandler、ParameterHandler、ResultSetHandler)处提供了简单易用的插 件扩展机制。
	  Mybatis对持久层的操作就是借助于四大核心对象。MyBatis支持用插件对四大核心对象进行拦截，对mybatis来说插件就是拦截器，
	  用来增强核心对象的功能，增强功能本质上是借助于底层的 动态代理实现的，换句话说，MyBatis中的四大对象都是代理对象
	2.MyBatis所允许拦截的方法如下：
		（1）执行器Executor (update、query、commit、rollback等方法)；
		（2）SQL语法构建器StatementHandler (prepare、parameterize、batch、updates query等方 法)；
		（3）参数处理器ParameterHandler (getParameterObject、setParameters方法)；
		（4）结果集处理器ResultSetHandler (handleResultSets、handleOutputParameters等方法)；
	3.在四大对象创建的时候：
		1、每个创建出来的对象不是直接返回的，而是interceptorChain.pluginAll(parameterHandler);
		2、获取到所有的Interceptor (拦截器)(插件需要实现的接口)；调用 interceptor.plugin(target);返回 target 包装后的对象
		3、插件机制，我们可以使用插件为目标对象创建一个代理对象；AOP (面向切面)我们的插件可以为四大对象创建出代理对象，代理对象就可以拦截到四大对象的每一个执行；
	4.源码刨析：
		
		
		
		


