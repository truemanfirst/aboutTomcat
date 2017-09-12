# aboutTomcat
talk some about tomcat resource
一、github账号已经很久了，但是一直潜水，希望也能够分享一些日常学习的心得，多多交流才会更快的成长；
二、今天主要分析一些tomcat的学习心得后期会做进一步的补充；
三、下面就是主菜了：
1、首先讲一下tomcat总体设计，了解一下它为什么而存在，主要的作用是什么；
  tomcat一个主要的作用就是解析报文，也就是我们在浏览器打出url，访问后台，会传递一段编码的字符串，字符串以/r/n分割，每个位置都有着固定而可变化的内容，也就是我们常说的协议，支持有HTTP、HTTPS、AJP协议等，这一协议是应用层的协议，主要便于浏览器和后台的交互，也可以说是一种标准，这一协议主要依于tcp协议，这是一个传输层的协议，就是字符串的编码和解码，浏览器通过url访问传递找到固定的ip将报文传给对应的计算机的对应端口，tomcat通过监听固定端口获取相应的报文，之后进行报文解析处理，tomcat通过http协议作为标准将获取到的报文填充到Request类当中，以便于后期的处理，Request请求分为动态请求和静态请求，动态请求会通过urlClassLoader类加载器获取对应类，找到对应的方法然后执行返回Response内容；静态请求会直接读取对应项目下webRoot目录里xml文件内容。
2、然后说一下tomcat的架构设计；
  tomcat通过Catalina类进行开始/关闭shell脚本的交互，由一个server封装，提供多个service服务，主要是基于多线程的设计方式，每个服务中提供两种结构httpConnector和container容器，httpConnector主要用于获取某一协议的连接，container则主要用于进行报文处理，而我们通常提到的集群和负载均衡就是在httpConnector处进行的处理，通过一个入口代理采用轮询或者按照权重比的方式将链接转发到对应的主机上来减轻服务器的cpu负担；
   container是一个容器类，通过统一的接口可以使得容器内部继续套用子容器； container主要包括Engine,host,Context,wrapper四个主要的容器；Engine包含Host和Context，在接收到请求以后会获取对应的虚拟主机Host，并获取相应的web应用的上下文context，每个请求都在是相应的上下文环境下进行处理的，tomcat通过映射的mapping找到对应的httpServlet，通过上下文中传入的信息参数执行操作；Wrapper是针对每个Servlet的Container，每个Servlet都有相应的Wrapper来管理。
   此外container还包含realm,manager,loader以及pipeline;realm主要用于权限的扩展用于获取权限和角色列表，manager主要用于cache缓存的管理，loader主要用于类的加载，jvm中主要包括三层类加载机制，bootstrap类加载，externsion类加载，application类加载；三者自上及下为父子关系，依次加载启动类，扩展类以及应用级别的类，而tomcat中class文件存储在WebApp/WEB-INF当中，需要自定义类加载器来加载到方法区的运行时常量池当中，tomcat中主要定义了五个自己的类加载器，Common类加载器，Catalina类加载器，Shared类加载器，WebApp类加载器以及Jsp类加载器分别用来加载（/common/,/server/,/shared/以及/WebApp/WEB-INF）包中的类和jsp文件，每一个Web应用对应着一个WebApp类加载器，每一个jsp文件对应着一个jsp类加载器;pipeline则为container通向servlet的管道，servlet中存放着开发者实现的service方法，完成调用后返回response，在response中返回json结果或者、xml（包括html，jsp等）内容;
3、浅谈一下Cookie的设置，Cookie可以通过response中设置key为Set-Cookie的值来设置，在返回带有Set-Cookie报文后，浏览器的下一次请求就会设置Cookie值为response的返回Set-Cookie的value值;众所周知浏览器访问后台时第一次servlet的访问会在后台会生成一个key值为jsessionid的键值对，返回作为浏览器的标记Cookie，我们可以通过jsessionid对应值在sessionManager当中获取其中存储的session值，所以我们可以通过session存储获取当前的用户信息，但是Android和ios访问后端的c/s方式不适用这种情况。用户信息存储在session当中虽然简单但是存在一个问题就是不能够统一进行管理，因为sessionid是存储在java堆的TLAB即线程本地分配缓存当中的，一个用户登录无法获取到其它浏览器当中是否有同一个用户进行了登录，所以无法进行用户登录的管理，踢出，限制账户数量等操作，此处就要引入缓存进行处理，例如redis，memcache等，将数据存储在统一的存储位置来进行处理。
4、关于realm权限的处理，目前使用较为多的是shiro权限管理；shiro提供了自己的sessionManager,cacheManager，我们可以通过自定义sessionDao进行session存储位置的改写，获取和删除；
