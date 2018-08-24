## server.xml解析
   
```
<Server> 顶层类元素，可包含多个service
        <Service> 顶层类元素，可包含一个Engine和多个Connector
            <Connector/> 链接类容器，代表通信接口
            <Engine> 容器元素，为Service处理客户请求,含多个Host
                <Host> 容器元素，为Host处理客户请求，含多个Context
                    <Context/> 容器元素，为Web应用处理客户请求
                </Host>
            </Engine>
        </Service>
    </Server>
```


## 一个java web应用在Tomcat中与一个Context对应，是一一对应关系。

    Java Web应该可以包含如下内容：
    Servlet
    JSP pages
    Java Classes
    static resources(HTML documents, pictures, etc.)
    Description Documents of Web  Applications.

## 1 - Tomcat Server的组成部分


    1.1 - Server
    A Server element represents the entire Catalina servlet container. (Singleton)
    （Server元素表示整个Catalina servlet容器。（singleton）单独一个）

    1.2 - Service
    A Service element represents the combination of one or more Connector components that share a single Engine
    Service是这样一个集合：它由一个或者多个Connector组成，以及一个Engine，负责处理所有Connector所获得的客户请求

    1.3 - Connector
    一个Connector将在某个指定端口上侦听客户请求，并将获得的请求交给Engine来处理，从Engine处获得回应并返回客户
    TOMCAT有两个典型的Connector，一个直接侦听来自browser的http请求，一个侦听来自其它WebServer的请求
    Coyote Http/1.1 Connector 在端口8080处侦听来自客户browser的http请求
    Coyote JK2 Connector 在端口8009处侦听来自其它WebServer(Apache)的servlet/jsp代理请求

    1.4 - Engine
    The Engine element represents the entire request processing machinery associated with a particular Service
    It receives and processes all requests from one or more Connectors
    and returns the completed response to the Connector for ultimate transmission back to the client
    Engine下可以配置多个虚拟主机Virtual Host，每个虚拟主机都有一个域名
    当Engine获得一个请求时，它把该请求匹配到某个Host上，然后把该请求交给该Host来处理
    Engine有一个默认虚拟主机，当请求无法匹配到任何一个Host上的时候，将交给该默认Host来处理

    1.5 - Host
    代表一个Virtual Host，虚拟主机，每个虚拟主机和某个网络域名Domain Name相匹配
    每个虚拟主机下都可以部署(deploy)一个或者多个Web App，每个Web App对应于一个Context，有一个Context path
    当Host获得一个请求时，将把该请求匹配到某个Context上，然后把该请求交给该Context来处理
    匹配的方法是“最长匹配”，所以一个path==""的Context将成为该Host的默认Context
    所有无法和其它Context的路径名匹配的请求都将最终和该默认Context匹配

    1.6 - Context
    一个Context对应于一个Web Application，一个Web Application由一个或者多个Servlet组成
    Context在创建的时候将根据配置文件$CATALINA_HOME/conf/web.xml和$WEBAPP_HOME/WEB-INF/web.xml载入Servlet类
    当Context获得请求时，将在自己的映射表(mapping table)中寻找相匹配的Servlet类
    如果找到，则执行该类，获得请求的回应，并返回
    
    
## 2 - Tomcat Server的结构图

![结构图](http://docs.huihoo.com/apache/tomcat/heavyz/01-startup.gif)


## 3 - 配置文件$CATALINA_HOME/conf/server.xml的说明


```
<!-- 启动Server  
     在端口8005处等待关闭命令  
     如果接受到"SHUTDOWN"字符串则关闭服务器  
     -->  
  
<Server port="8005" shutdown="SHUTDOWN" debug="0">  
  
  
  <!-- Listener ???  
       目前没有看到这里  
       -->  
  
  <Listener className="org.apache.catalina.mbeans.ServerLifecycleListener" debug="0"/>  
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" debug="0"/>  
  
  
  <!-- Global JNDI resources
       全球命名资源  
       -->  
  
  <GlobalNamingResources>  
    <Resource name="jdbc/mydb" type="javax.sql.DataSource" password="mypwd" driverClassName="com.microsoft.jdbc.sqlserver.SQLServerDriver" maxIdle="30" maxWait="5000" validationQuery="select 1" username="sa" url="jdbc:microsoft:sqlserver://localhost:1433;DatabaseName=mydb" maxActive="200"/>
  </GlobalNamingResources>  
  <!-- 
      maxIdle，最大空闲数，数据库连接的最大空闲时间。超过空闲时间，数据库连接将被标记为不可用，然后被释放。设为0表示无限制。
      MaxActive，连接池的最大数据库连接数。设为0表示无限制。
      maxWait ，最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。 
   -->
  
  
  <!-- Tomcat的Standalone Service  
       Service是一组Connector的集合  
       它们共用一个Engine来处理所有Connector收到的请求  
       -->  
  
  <Service name="Tomcat-Standalone">  
  
      <!--The connectors can use a shared executor, you can define one or more named thread pools
      初始化线程并发
      可以配置线程池，配置线程池后  下面Connector 的配置连接数无效
      -->
    <!--
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
    -->
    <!-- 
        maxThreads :Tomcat 使用线程来处理接收的每个请求，这个值表示 Tomcat 可创建的最大的线程数，默认值是 200
    
        minSpareThreads：最小空闲线程数，Tomcat 启动时的初始化的线程数，表示即使没有人使用也开这么多空线程等待，默认值是 10。
        
        maxSpareThreads：最大备用线程数，一旦创建的线程超过这个值，Tomcat 就会关闭不再需要的 socket 线程。
    -->
  
    <!-- Coyote HTTP/1.1 Connector  
         className : 该Connector的实现类是org.apache.coyote.tomcat4.CoyoteConnector  
         port : 在端口号8080处侦听来自客户browser的HTTP1.1请求  
         minProcessors : 该Connector先创建5个线程等待客户请求，每个请求由一个线程负责  
         maxProcessors : 当现有的线程不够服务客户请求时，若线程总数不足75个，则创建新线程来处理请求  
         acceptCount : 当现有线程已经达到最大数75时，为客户请求排队  
                       当队列中请求数超过100时，后来的请求返回Connection refused错误  
         redirectport : 当客户请求是https时，把该请求转发到端口8443去  
         其它属性略  
         -->  
  
    <Connector className="org.apache.coyote.tomcat4.CoyoteConnector"   
               port="8080"   
               minProcessors="5" maxProcessors="75" acceptCount="100"   
               enableLookups="true"   
               redirectPort="8443"   
               debug="0"   
               connectionTimeout="20000"   
               useURIValidationHack="false"   
               disableUploadTimeout="true" />  
  
  
    <!-- Engine用来处理Connector收到的Http请求  
         它将匹配请求和自己的虚拟主机，并把请求转交给对应的Host来处理  
         默认虚拟主机是localhost  
         -->  
  
    <Engine name="Standalone" defaultHost="localhost" debug="0">  
      
  
      <!-- 日志类，目前没有看到，略去先 -->  
  
      <Logger className="org.apache.catalina.logger.FileLogger" .../>  
  
      <!-- Realm，目前没有看到，略去先 -->  
  
      <Realm className="org.apache.catalina.realm.UserDatabaseRealm" .../>  
  
  
      <!-- 虚拟主机localhost  
           appBase : 该虚拟主机的根目录是webapps/  
           它将匹配请求和自己的Context的路径，并把请求转交给对应的Context来处理  
           -->  
  
      <Host name="localhost" debug="0" appBase="webapps" unpackWARs="true" autoDeploy="true">  
        
  
        <!-- 日志类，目前没有看到，略去先 -->  
  
        <Logger className="org.apache.catalina.logger.FileLogger" .../>  
        
  
        <!-- Context，对应于一个Web App  
             path : 该Context的路径名是""，故该Context是该Host的默认Context  
             docBase : 该Context的根目录是webapps/mycontext/  
             -->  
  
        <Context path="" docBase="mycontext" debug="0"/>  
          
  
        <!-- 另外一个Context，路径名是/wsota -->  
  
        <Context path="/wsota" docBase="wsotaProject" debug="0"/>  
               
          
      </Host>  
        
    </Engine>  
  
  </Service>  
  
</Server>  
  
  
<!----------------------------------------------------------------------------------------------->  

 

4 - Context的部署配置文件web.xml的说明
一个Context对应于一个Web App，每个Web App是由一个或者多个servlet组成的
当一个Web App被初始化的时候，它将用自己的ClassLoader对象载入“部署配置文件web.xml”中定义的每个servlet类
它首先载入在$CATALINA_HOME/conf/web.xml中部署的servlet类
然后载入在自己的Web App根目录下的WEB-INF/web.xml中部署的servlet类
web.xml文件有两部分：servlet类定义和servlet映射定义
每个被载入的servlet类都有一个名字，且被填入该Context的映射表(mapping table)中，和某种URL PATTERN对应
当该Context获得请求时，将查询mapping table，找到被请求的servlet，并执行以获得请求回应
分析一下所有的Context共享的web.xml文件，在其中定义的servlet被所有的Web App载入
[html] view plain copy
<!----------------------------------------------------------------------------------------------->  
  
  
<web-app>  
  
  
  <!-- 概述：  
       该文件是所有的WEB APP共用的部署配置文件，  
       每当一个WEB APP被DEPLOY，该文件都将先被处理，然后才是WEB APP自己的/WEB-INF/web.xml  
       -->  
  
  
  
  <!--  +-------------------------+  -->  
  <!--  |    servlet类定义部分    |  -->  
  <!--  +-------------------------+  -->  
  
    
  
  <!-- DefaultServlet  
       当用户的HTTP请求无法匹配任何一个servlet的时候，该servlet被执行  
       URL PATTERN MAPPING : /  
       -->  
  
    <servlet>  
        <servlet-name>default</servlet-name>  
        <servlet-class>  
          org.apache.catalina.servlets.DefaultServlet  
        </servlet-class>  
        <init-param>  
            <param-name>debug</param-name>  
            <param-value>0</param-value>  
        </init-param>  
        <init-param>  
            <param-name>listings</param-name>  
            <param-value>true</param-value>  
        </init-param>  
        <load-on-startup>1</load-on-startup>  
    </servlet>  
  
  
  <!-- InvokerServlet  
       处理一个WEB APP中的匿名servlet  
       当一个servlet被编写并编译放入/WEB-INF/classes/中，却没有在/WEB-INF/web.xml中定义的时候  
       该servlet被调用，把匿名servlet映射成/servlet/ClassName的形式  
       URL PATTERN MAPPING : /servlet/*  
       -->  
  
    <servlet>  
        <servlet-name>invoker</servlet-name>  
        <servlet-class>  
          org.apache.catalina.servlets.InvokerServlet  
        </servlet-class>  
        <init-param>  
            <param-name>debug</param-name>  
            <param-value>0</param-value>  
        </init-param>  
        <load-on-startup>2</load-on-startup>  
    </servlet>  
  
  
  <!-- JspServlet  
       当请求的是一个JSP页面的时候（*.jsp）该servlet被调用  
       它是一个JSP编译器，将请求的JSP页面编译成为servlet再执行  
       URL PATTERN MAPPING : *.jsp  
       -->  
  
    <servlet>  
        <servlet-name>jsp</servlet-name>  
        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>  
        <init-param>  
            <param-name>logVerbosityLevel</param-name>  
            <param-value>WARNING</param-value>  
        </init-param>  
        <load-on-startup>3</load-on-startup>  
    </servlet>  
  
  
  
  <!--  +---------------------------+  -->  
  <!--  |    servlet映射定义部分    |  -->  
  <!--  +---------------------------+  -->  
  
      
    <servlet-mapping>  
        <servlet-name>default</servlet-name>  
        <url-pattern>/</url-pattern>  
    </servlet-mapping>  
  
    <servlet-mapping>  
        <servlet-name>invoker</servlet-name>  
        <url-pattern>/servlet/*</url-pattern>  
    </servlet-mapping>  
  
    <servlet-mapping>  
        <servlet-name>jsp</servlet-name>  
        <url-pattern>*.jsp</url-pattern>  
    </servlet-mapping>  
  
  
  <!--  +------------------------+  -->  
  <!--  |    其它部分，略去先    |  -->  
  <!--  +------------------------+  -->  
  
    ... ... ... ...  
  
</web-app>  
 
```
## 5 - Tomcat Server处理一个http请求的过程

假设来自客户的请求为：
http://localhost:8080/wsota/wsota_index.jsp
1) 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector获得
2) Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应
3) Engine获得请求localhost/wsota/wsota_index.jsp，匹配它所拥有的所有虚拟主机Host
4) Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）
5) localhost Host获得请求/wsota/wsota_index.jsp，匹配它所拥有的所有Context
6) Host匹配到路径为/wsota的Context（如果匹配不到就把该请求交给路径名为""的Context去处理）
7) path="/wsota"的Context获得请求/wsota_index.jsp，在它的mapping table中寻找对应的servlet
8) Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类
9) 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
10)Context把执行完了之后的HttpServletResponse对象返回给Host
11)Host把HttpServletResponse对象返回给Engine
12)Engine把HttpServletResponse对象返回给Connector
13)Connector把HttpServletResponse对象返回给客户browser