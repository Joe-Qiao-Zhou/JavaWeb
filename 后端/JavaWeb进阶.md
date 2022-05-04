# Tomcat

- SUN公司指定JavaEE规范：JDBC，JNDI，EJB，RMI，JSP，Servlets，XML，JMS，Java IDL，JTS，JTA，JavaMail，JAF
- Apache的Tomcat服务器只实现了Serlvet和JSP，因此被称为轻量级服务器或Servlet/JSP容器
- 通过**IP+域名**找到一个服务器上部署的**多个虚拟主机**中的某一个
- **Tomcat服务器 = Web服务器 + Servlet/JSP容器（Web容器）**，浏览器只能解析静态资源，容器用来将动态资源转换为静态资源
  - 动态资源：可拼装+随时间变化+每名用户访问内容不同，由服务器组装后通过HTTP返回

# Cookie与Session

- 从打开一个浏览器访问某个站点到关闭浏览器的整个过程称为一次会话

## 为什么需要会话机制

- 一个会话可以帮助服务器断定一个客户端，当服务器无法断定客户端时（session失效或cookie失效），一次会话就结束了
- 会话机制最主要的目的是**帮助服务器记住客户端状态**（标识用户，跟踪状态），目前CS通讯都是通过无状态的HTTP协议，会话跟踪技术是对HTTP的一种扩展

## Cookie

- Cookie其实是一份小数据，是服务器响应给客户端，并且**存储在客户端**的一份小数据，下次客户端访问服务器时，会**自动带上**这个Cookie，服务器通过Cookie就可以区分客户端，相当于吃饭取位置的小票
- 两种类型
  - 会话Cookie
    1. 服务器new任意数量的Cookie，通过response的addCookie()响应给浏览器
    2. 浏览器在HTTP响应头中获取Cookie
    3. 客户端第二次请求同一个服务器时会在请求头中带上Cookie
    4. 这种方式传给客户端的cookie会被保存在浏览器的内存中，随着客户端的关闭而消失
  - 持久性Cookie
    1. 通过setMaxAge()设置持久化时间，存储在磁盘上

## Session

- 出于安全性和传输效率而提出，Session在服务端，不再将数据放在头里传输，而只给客户端传一个JSESSIONID(其实也是一个Cookie)，真正的数据在服务端的Session中，下载访问服务器时带上这个ID，服务器就能在自己这里找到对应的Session，取出用户信息
- 整个请求响应的过程就是序列化和反序列化的过程
  - 服务器将Java对象序列化为JSON
  - 网络不能传递对象但能传递字符串
  - 浏览器得到JSON后反序列化为JS对象
  - 服务器与服务器之间也需要JSON作为中介

- Session序列化：将Session序列化到磁盘中，等服务器重启完毕后重新读回内存；如果Session中的对象没有实现序列化接口便无法被序列化到硬盘，会直接从内存中消失
- Session钝化与活化：当一个Session长时间无人访问，便利用序列化技术将其钝化到磁盘上以减少内存占用；当Session再次被访问时就会利用反序列化技术活化该Session
- 与服务器关闭时的序列化不同点
  - 每个Session单独一个文件而不是打包成一个文件
  - 即使活化到内存，磁盘上的文件也不消失

# Listener

- 分为6个常规监听器和2个感知监听器，常规监听器分别对应JavaWeb三大域对象，生命周期监听用来监听域对象的创建与销毁，属性监听用来监听域对象的get/setAttribute()
  - ServletContext
    - ServletContextListener（生命周期监听）
    - ServletContextAttributeListener（属性监听）
  - HttpSession
    - HttpSessionListener（生命周期监听）
    - HttpSessionAttributeListener（属性监听）
  - ServletRequest
    - ServletRequestListener（生命周期监听）
    - ServletRequestAttributeListener（属性监听）
  - HttpSessionBindingListener&HttpSessionActivationListener，均与Session相关
- 几个问题
  1. 三大生命周期监听器，分别在何时被创建、销毁？
     - ServletContextListener
       - 1.**项目启动时**，监听到ServletContext对象被创建
       - 6.项目关停，监听到ServletContext被销毁
     - ServletRequestListener
       - 2.**每一次浏览器发起请求**Tomcat都会创建一个Request，被监听到
       - 4.请求结束，Request销毁，被监听到
     - HttpSessionListener
       - 3.**如果**Servlet中调用了request.getSession()，则Tomcat会创建Session（如果根据JSESSIONID找不到对应的），这会被HttpSessionListener监听到
       - 5.用户30分钟未访问，Session过期销毁，被监听到
  2. 访问一个Servlet，HttpSessionListener一定会触发么？（访问Servlet，Session一定会创建么）
     - 只有当在Servlet中调用request.getSession()，且根据JSESSIONID找不到对应的Session时，才会创建新的Session对象，才会被监听到
  3. 三大属性监听器何时触发？
     - get/setAttribute()时

![图片.png](https://cdn.nlark.com/yuque/0/2021/png/1057015/1616975067612-82ad1a07-fea7-47b3-b724-ed9603bd1d7e.png?x-oss-process=image%2Fresize%2Cw_1045%2Climit_0)

- Listener涉及到观察者模式，包括被监听对象、事件和监听器对象，当被监听对象执行某个事件时，会**主动告诉**Listener，让它执行对应操作

# Filter

- 配置Filter时可以配置4种拦截方式：REQUEST、FORWARD、INCLUDE、ERROR，这4种均与request有关，而REDIRECT则与response相关

![图片.png](https://cdn.nlark.com/yuque/0/2021/png/1057015/1616215348647-7af453c9-8347-47fc-9063-d51d9f44b8f9.png)

- 两类的最大区别：重定向会使浏览器发送2次请求，而其他的则是服务器内部的1次请求，不涉及浏览器

![图片.png](https://cdn.nlark.com/yuque/0/2021/png/1057015/1616215433769-64f85a1e-342f-4a68-900f-b1261392ba0f.png)

# Servlet

- Web服务器负责接收请求和响应请求，而处理请求的逻辑不同，抽取成Servlet，继续抽取成Service和Dao
- HTTP请求到了Tomcat后，Tomcat通过字符串解析将请求头、URL、请求参数都封装进了Request对象中
- Tomcat传给Servlet的Response对象是空的，Servlet处理后得到的结果会写入Response内部的缓冲区，Tomcat在Servlet处理结束后，拿到Response，遍历里面的信息，组装成HTTP响应发给客户端
- 如何写一个Servlet？不用实现javax.servlet接口，不用继承GenericServlet抽象类，只需继承HttpServlet并重写doGet()/doPost()
- 父类把能写的逻辑都写完，把不确定的业务代码抽成一个方法并调用它，当子类重写该方法，整个业务代码就活了，这就是模板方法模式

## ServletContext

- ServletContext直接关系到SpringIOC容器的初始化，Servlet映射规则与SpringMVC密切相关

- 服务器为每个应用创建一个ServletContext对象，在服务器启动时创建，在服务器关闭时销毁
- ServletContext对象的作用：在整个Web应用的动态资源（Servlet/JSP）之间共享数据，例如在AServlet中向ServletContext对象保存一个值，然后在BServlet中就可以获取这个值

![图片.png](https://cdn.nlark.com/yuque/0/2021/png/1057015/1616215028555-e1edf2df-4663-48af-bcd4-62f64e4ba2c2.png)

- JavaWeb中一共有4个域对象用来装载共享数据，都有get/setAttribute()
  1. ServletContext域（Servlet间共享数据）
  2. Session域（一次会话间共享数据，也可以理解为多次请求间共享数据）
  3. Request域（同一次请求共享数据）
  4. Page域（JSP页面内共享数据）

- 与web.xml的对应关系
  - 整个web.xml对应ServletContext
  - \<servlet>对应一个Servlet
  - \<init-param>对应初始化所需要的servletConfig参数

- 获取ServletContext的5种方法
  1. ServletConfig#getServletContext()
  2. GenericServlet#getServletContext()
  3. HttpSession#getServletContext()
  4. HttpServletRequest#getServletContext()
  5. ServletContextEvent#getServletContext()

## Servlet映射器

- 每个URL交给哪个Servlet处理由映射器决定，是Tomcat中的Mapper类，提供精确匹配、前缀匹配、扩展名匹配、欢迎列表资源匹配，如果都不匹配则交给DefaultServlet，无论是静态资源、Servlet还是JSP，Tomcat最终都会交给对应的Servlet类去处理
- 如果拦截路径配置为/*，则所有Servlet都会被短路，这就导致JSP无法被编译成Servlet输出HTML片段（JspServlet短路），HTML/CSS/JS/PNG等资源无法获取（DefaultServlet短路）
- 如果拦截路径配置为/，JSP不会被拦截，但是静态资源依然被拦截
- Tomcat的conf/web.xml是Tomcat的全局配置，里面配置了JspServlet和DefaultServlet，而每个项目都会继承这个web.xml中的配置，拦截路径默认为.jsp和/

# POST请求

- 常见的3种
  1. 普通表单
     - 默认编码方式为`enctype="application/x-www-form-urlencoded"`，服务器会自动分割参数并存入Map中，可以通过request.getParameter()获取参数，但是无法上传文件
  2. 文件表单
     - 编码方式为`enctype="multipart/form-data"`，多部件传输请求，将表单的每一项作为一个组件单独发送，每个组件都包含请求头、空行和请求体
     - 服务器不再解析参数，直接放入request的inputStream中，需要自己获取
     - Servlet的commons-fileupload依赖帮助接收File
     - SpringMVC集成了依赖，在请求到达Controller方法前，已经将文件封装到MultipartFile组件中
  3. JSON提交
     - raw+JSON的形式最常用，即直接发送数据，后端将其包装为POJO
     - 为了接收JSON请求，后端必须加上@Requestbody：**SpringMVC将会用jackson解析请求体中参数，其他非JSON格式的参数会直接抛异常**

![img](https://cdn.nlark.com/yuque/0/2020/png/432628/1579439206460-b5da2835-c3e7-4c05-bc91-ffdff51f827a.png)

# POJO与JSON

![image.png](https://cdn.nlark.com/yuque/0/2020/png/432628/1607249238783-2c299a8f-258e-490b-a8c5-e29b9e92ba73.png?x-oss-process=image%2Fresize%2Cw_527%2Climit_0)

- JS对象和Java对象分别存活于浏览器和服务器的内存中，而JSON是特定格式的字符串，承担网络传输的角色
- 