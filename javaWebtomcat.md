#### statement
	select * from user where username='sb' and password='nuini';
#### preparedStatement
	select * from user where username=? and password=?;
#### 连接池
	程序中用来共享资源的容器，线程池、常量池
	数据库连接池就是用来存放数据库连接的容器
#### 如何自己实现一个连接池??

#### 1.3 servlet
	静态web资源（不同的条件下访问的效果都是一样的 html/css/js/flash）
	动态web资源（不同访问条件，显示不同的数据，称之为动态web servlet/jsp/.net/c#/php/asp）
####jsp和servlet的异同
	jsp 都可以处理请求，做出响应
	1.servlet，java程序，适合逻辑处理，不适合展示数据
	2.servlet没有内置对象，jsp有九大内置对象
	3.在MVC中servlet属于控制器controller，jsp属于视图view
####servlet 调用过程
	请求->解析主机->解析应用->找到访问的资源——>第一次访问时创建servlet对象调用servlet的init（）方法初始化；
	->服务器关闭或应用被移除容器时调用destory()方法销毁；
####转发与重定向的区别
	转发服务器内部资源跳转 
	重定向可以是同一个应用内部也可以不是同一个虚拟机之间跳转；
	转发前后是同一个request对象，浏览器地址栏前后不会改变
####四大域特征
	request域 生命周期请求开始时创建，结束后销毁
	session域 
####jsp九大隐式对象
####过滤器Filter（拦截器）javaweb3大组件之一
	需要在web.xml中注册
	对用户访问进行拦截（权限控制等）
	1.写一个类实现Filter接口；实现doFilter方法
####监听器，用来监听java对象的java程序
####cookie 和 session 会话技术
	会话 浏览器和服务器之间多次请求和响应成为一次会话；
	 session销毁 超时（过期自然销毁）、自杀（退出登录，调用invalidate方法）、意外身亡（服务器关闭）
###tomcat调优
1、工作方式选择
为了提升性能，首先就要对代码进行动静分离，让 Tomcat 只负责 jsp 文件的解析工作。如采用 Apache 和 Tomcat 的整合方式，他们之间的连接方案有三种选择，JK、http_proxy 和 ajp_proxy。相对于 JK 的连接方式，后两种在配置上比较简单的，灵活性方面也一点都不逊色。但就稳定性而言不像JK 这样久经考验，所以建议采用 JK 的连接方式。
2、Connector 连接器的配置
之前文件介绍过的 Tomcat 连接器的三种方式： bio、nio 和 apr，三种方式性能差别很大，apr 的性能最优， bio 的性能最差。而 Tomcat 7 使用的 Connector 默认就启用的 Apr 协议，但需要系统安装 Apr 库，否则就会使用 bio 方式。
3、配置文件优化
配置文件优化其实就是对 server.xml 优化，可以提大大提高 Tomcat 的处理请求的能力，下面我们来看 Tomcat 容器内的优化。
默认配置下，Tomcat 会为每个连接器创建一个绑定的线程池（最大线程数 200），服务启动时，默认创建了 5 个空闲线程随时等待用户请求。
首先，打开 ${TOMCAT_HOME}/conf/server.xml，搜索【<Executor name="tomcatThreadPool"】，开启并调整为
maxThreads :Tomcat 使用线程来处理接收的每个请求，这个值表示 Tomcat 可创建的最大的线程数，默认值是 200。
minSpareThreads：最小空闲线程数，Tomcat 启动时的初始化的线程数，表示即使没有人使用也开这么多空线程等待，默认值是 10。
maxSpareThreads：最大备用线程数，一旦创建的线程超过这个值，Tomcat 就会关闭不再需要的 socket 线程。
上边配置的参数，最大线程 500（一般服务器足以），要根据自己的实际情况合理设置，设置越大会耗费内存和 CPU，因为 CPU 疲于线程上下文切换，没有精力提供请求服务了，最小空闲线程数 20，线程最大空闲时间 60 秒，当然允许的最大线程连接数还受制于操作系统的内核参数设置，设置多大要根据自己的需求与环境。当然线程可以配置在“tomcatThreadPool”中，也可以直接配置在“Connector”中，但不可以重复配置。
URIEncoding：指定 Tomcat 容器的 URL 编码格式，语言编码格式这块倒不如其它 WEB 服务器软件配置方便，需要分别指定。
connnectionTimeout： 网络连接超时，单位：毫秒，设置为 0 表示永不超时，这样设置有隐患的。通常可设置为 30000 毫秒，可根据检测实际情况，适当修改。
enableLookups： 是否反查域名，以返回远程主机的主机名，取值为：true 或 false，如果设置为false，则直接返回IP地址，为了提高处理能力，应设置为 false。
disableUploadTimeout：上传时是否使用超时机制。
connectionUploadTimeout：上传超时时间，毕竟文件上传可能需要消耗更多的时间，这个根据你自己的业务需要自己调，以使Servlet有较长的时间来完成它的执行，需要与上一个参数一起配合使用才会生效。
acceptCount：指定当所有可以使用的处理请求的线程数都被使用时，可传入连接请求的最大队列长度，超过这个数的请求将不予处理，默认为100个。
keepAliveTimeout：长连接最大保持时间（毫秒），表示在下次请求过来之前，Tomcat 保持该连接多久，默认是使用 connectionTimeout 时间，-1 为不限制超时。
maxKeepAliveRequests：表示在服务器关闭之前，该连接最大支持的请求数。超过该请求数的连接也将被关闭，1表示禁用，-1表示不限制个数，默认100个，一般设置在100~200之间。
compression：是否对响应的数据进行 GZIP 压缩，off：表示禁止压缩；on：表示允许压缩（文本将被压缩）、force：表示所有情况下都进行压缩，默认值为off，压缩数据后可以有效的减少页面的大小，一般可以减小1/3左右，节省带宽。
compressionMinSize：表示压缩响应的最小值，只有当响应报文大小大于这个值的时候才会对报文进行压缩，如果开启了压缩功能，默认值就是2048。
compressableMimeType：压缩类型，指定对哪些类型的文件进行数据压缩。
noCompressionUserAgents="gozilla, traviata"： 对于以下的浏览器，不启用压缩。
如果已经对代码进行了动静分离，静态页面和图片等数据就不需要 Tomcat 处理了，那么也就不需要配置在 Tomcat 中配置压缩了。
以上是一些常用的配置参数属性，当然还有好多其它的参数设置，还可以继续深入的优化，HTTP Connector 与 AJP Connector 的参数属性值，可以参考官方