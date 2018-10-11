# FtpClientPool
这是一个FTP客户端连接池，底层依赖apache的commons-net和commons-pool2，基于这二者做的个薄层封装，让我们可以在项目中以更简洁、高效的方式使用FtpClient。
# > 获取FtpClientPool
----
## 1.直接下载jar包

## 2.Maven依赖
```xml
<dependency>
  <groupId>com.itshidu.commons</groupId>
  <artifactId>FtpClientPool</artifactId>
  <version>1.0</version>
</dependency>
```
.
# > 如何使用
----
## 1.直接编码使用
```xml
//配置信息
FtpPoolConfig cfg = new FtpPoolConfig();
cfg.setHost("192.168.61.110");
cfg.setPort(21);
cfg.setUsername("ftpuser");
cfg.setPassword("123456");

FTPClientFactory factory = new FTPClientFactory(cfg);//对象工厂
FTPClientPool pool = new FTPClientPool(factory);//连接池对象
FtpClientUtils util = new FtpClientUtils(); //工具对象

FTPClient c = pool.borrowObject();//从池子中借一个FTPClient对象
util.mkdirs(c, "/data/imgs"); //在FTP的工作目录下创建多层目录
InputStream in = new FileInputStream("D:/001.jpg"); //读取一个本地文件
util.store(c, in, "/data/imgs/2018/09/29", "main.jpg");//上传到FTP服务器
util.retrieve(c, "/data/imgs/2018/09/29/main.jpg", new FileOutputStream("F:/002.jpg"));//从FTP服务器取回文件
util.delete(c, "/data/imgs/2018/09/29/main.jpg"); //删除FTP服务器中的文件
pool.returnObject(c);//把对象归还给池子

```


## 2.集成到Spring中使用
在你的spring主配置文件中引入spring-ftpClientPool.xml配置文件
	    示例如下：   
      
	 <import resource="classpath:spring-ftpClientPool.xml"/>
   
   spring-ftpClientPool.xml 配置如下：
   
   ```
   <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:cache="http://www.springframework.org/schema/cache"
	xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context-4.3.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
          http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
          http://www.springframework.org/schema/cache
          http://www.springframework.org/schema/cache/spring-cache-4.3.xsd">
    
    <beans>
      <!-- ftp连接池配置参数 -->
     <bean id="ftpPoolConfig" class="com.tingcream.ftpClientPool.config.FtpPoolConfig">
           <property name="host" value="${ftp.host}"/>
           <property name="port" value="${ftp.port}"/>
           <property name="username" value="${ftp.username}"/>
           <property name="password" value="${ftp.password}"/>
           
           <property name="connectTimeOut" value="${ftp.connectTimeOut}"/>
           <property name="controlEncoding" value="${ftp.controlEncoding}"/>
           <property name="bufferSize" value="${ftp.bufferSize}"/>
           <property name="fileType" value="${ftp.fileType}"/>
           <property name="dataTimeout" value="${ftp.dataTimeout}"/>
           <property name="useEPSVwithIPv4" value="${ftp.useEPSVwithIPv4}"/>
           <property name="passiveMode" value="${ftp.passiveMode}"/>
           
           <property name="blockWhenExhausted" value="${ftp.blockWhenExhausted}"/>
           <property name="maxWaitMillis" value="${ftp.maxWaitMillis}"/>
           <property name="maxTotal" value="${ftp.maxTotal}"/>
           <property name="maxIdle" value="${ftp.maxIdle}"/>
           <property name="testOnBorrow" value="${ftp.testOnBorrow}"/>
           <property name="testOnReturn" value="${ftp.testOnReturn}"/>
           <property name="testOnCreate" value="${ftp.testOnCreate}"/>
           <property name="testWhileIdle" value="${ftp.testWhileIdle}"/>
     </bean>
      <!-- ftp客户端工厂 -->
     <bean id="ftpClientFactory" class="com.tingcream.ftpClientPool.core.FTPClientFactory">
          <property name="ftpPoolConfig" ref="ftpPoolConfig"></property>
     </bean>
     
     <!-- ftp客户端连接池对象 -->
     <bean id="ftpClientPool" class="com.tingcream.ftpClientPool.core.FTPClientPool">
         <constructor-arg index="0" ref="ftpClientFactory"></constructor-arg>
     </bean> 
    
     <!-- ftp客户端辅助bean-->
      <bean id="ftpClientHelper" class="com.tingcream.ftpClientPool.client.FTPClientHelper">
           <property name="ftpClientPool"  ref="ftpClientPool"/>
      </bean>
     
</beans>
 ```
  
  
 
3、在你的 spring主配置文件中新载入一个ftpPoolConfig.properties配置
	    示例如下：
      
	   <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"> 
		  <property name="locations"> 
		    <list> 
		      <value>classpath:jdbc.properties</value> 
		      <value>classpath:redis.properties</value> 
		       <!--  更多项  -->
		       
		       <!--注意新加入此项  -->
		      <value>classpath:ftpPoolConfig.properties</value> 
		      
		    </list> 
		  </property> 
		</bean>
4、在你的工程classpath中新加ftpPoolConfig.properties属性配置文件

示例：  

```
ftp.host=127.0.0.1
ftp.port=21
ftp.username=ftptest
ftp.password=123456

 
#ftp 连接超时时间 毫秒
ftp.connectTimeOut=5000
#控制连接  字符编码
ftp.controlEncoding=UTF-8
#缓冲区大小
ftp.bufferSize=1024
#传输文件类型   2表二进制
ftp.fileType=2
#数据传输超时 毫秒
ftp.dataTimeout=120000
#
ftp.useEPSVwithIPv4=false
#是否启用ftp被动模式
ftp.passiveMode=true


#连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true
ftp.blockWhenExhausted=true
#最大空闲等待时间  毫秒
ftp.maxWaitMillis=3000
#最大连接数
ftp.maxTotal=10
#最大空闲连接数
ftp.maxIdle=10
#最小空闲连接数
ftp.minIdle=2
#申请连接时 检测是否有效
ftp.testOnBorrow=true
#返回连接时 检测是否有效
ftp.testOnReturn=true
#创建连接时 检测是否有效
ftp.testOnCreate=true
#空闲时检测连接是否有效  
ftp.testWhileIdle=true
```

5、在你的spring工程中如何使用此模块(ftpClientPool)

	    示例1:           
	    @Resource  
	    private  FTPClientHelper  ftpClientHelper ;// 注入ftp客户端helper对象使用
      
	  示例2:    
	    @Resource
	    private   FTPClientPool  ftpClientPool;// 注入 ftp客户端连接池对象使用
	  示例3：     你也可以同时注入上面二者
	  
Config的属性继承自，扩展属性如下：
```
private String host;// 主机名
private int port = 21;// 端口
private String username;// 用户名
private String password;// 密码
private int connectTimeOut = 5000;// ftp 连接超时时间 毫秒
private String controlEncoding = "utf-8";
private int bufferSize = 1024;// 缓冲区大小
private int fileType = 2;// 传输数据格式 2表binary二进制数据
private int dataTimeout = 120000;
private boolean useEPSVwithIPv4 = false;
private boolean passiveMode = true;// 是否启用被动模式
```
