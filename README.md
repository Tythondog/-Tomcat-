# -Tomcat-
@[TOC]
# 写在前面
作为一个刚接触Tomcat的萌新，觉得这个东西还有很好玩的。在写了一两个小网页之后，刚好大哥突然来问我能不能实现通过扫二维码访问特定网页的功能。我一想，这不就是要通过互联网访问本地的Tomcat服务器吗？抱着为大哥解决问题（主要是能更方便地把自己做的一些沙雕网页分享出去）的态度，我觉得这个问题很有解决的必要，于是经过大半天的摸索，有了这篇文章。

## 1.安装Tomcat
Tomcat是一款基于Java的服务器软件，可以很方便地集成到IDEA中，抛开Java不谈，它也是一款很优秀的软件。其安装方式也很简单，下载压缩包，直接解压到指定文件夹即可链接，附上Windows版64位的Tomcat8压缩包的百度云链接： [点击下载](https://pan.baidu.com/s/1I0hppToLfbUxwae_4v66NA)，提取码：ye3x。

其他版本参见官网：[https://tomcat.apache.org/download-80.cgi](https://tomcat.apache.org/download-80.cgi)

下载并解压到指定文件夹，Tomcat就安装好了，其目录结构如下图所示：
![Tomcat文件结构](https://img-blog.csdnimg.cn/20200331203154553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70#pic_center)
其中本次会用到的文件夹有：bin，conf，webapps，各文件夹功能如下：
 1. **bin文件夹**：存放可执行文件，tomcat的启动与正常关闭的文件就存放于此。![bin文件夹](https://img-blog.csdnimg.cn/20200331203936368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70#pic_center)
2. **conf文件夹**：存放配置文件，本次需要修改的主要有：logging.properties，server.xml。
![conf文件夹](https://img-blog.csdnimg.cn/20200331204911705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70#pic_center)
3. **webapps文件夹**：存放项目内容，未改动时存在一个默认项目。

## 2.安装过程中可能出现的问题
1. **闪退。** 原因有以下2种：1）未正确配置JAVA_HOME环境变量，由于Tomcat是基于java开发的，所以必须正确安装配置JDK环境；2）其它程序或者存在已经在运行的Tomcat进程占用了端口，Tomcat默认使用8080端口，如果其他进程占用该端口，会导致其无法正确运行。可通过杀掉占用8080端口的进程或者修改配置文件解决：conf-->server.xml-->
	```bash
   	# port即为Tomcat默认使用的端口号，可选范围为0-65535，但是有很一些端口为其他进程默认使用的，
   	# 如mysql默认端口号为3306，HTTP默认使用80端口等，因此不建议随意修改
	<Connector port="8080" protocol="HTTP/1.1"
	               connectionTimeout="20000"
	               redirectPort="8443" 
				   URIEncoding="UTF-8" />
	```
2. **乱码。** 如果出现如下图所示的乱码：![乱码](https://img-blog.csdnimg.cn/20200331220423884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70#pic_center)
则是由于Tomcat默认使用UTF-8编码方式，而Windows默认使用GBK编码。因此，修改conf-->logging.properties

	```bash
	java.util.logging.ConsoleHandler.encoding = UTF-8
	# 将上面的UTF-8修改为GBK即可
	java.util.logging.ConsoleHandler.encoding = GBK
	```
## 3.访问本地项目
完成以上操作后，Tomcat的准备工作已经就绪，可以开始访问本地项目。双击运行bin目录下的startup.bat运行，然后浏览器中访问以下地址：

```bash
# 如果没有修改默认端口号，访问
http://127.0.0.1:8080
# 或
http://localhost:8080
# 如果修改了默认端口号，将以上地址中的8080替换为自定义的端口号即可
```
访问结果如下图所示，为Tomcat自带的默认页面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331223104546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70#pic_center)
接下来创建自己的项目。以一个简单的项目为例，如果你的本地项目为F:\test文件夹，文件夹下有一些html文件，如HelloWorld.html等。你想要通过Tomcat访问test文件夹下的内容，可以按照如下方式配置：
1. **配置项目路径。** 在server.xml文件中，找到Host标签，在其中增加Context标签。其中docBase存放所要添加项目的绝对路径，如在本例中应该为"F:\test"；path存放虚拟路径，即访问该项目时所使用的名称，用来代替文件夹名“test”，本例使用“/web”，如下所示: 

	```bash
	# Host标签以<Host>开始，</Host>结束，中间为标签体内容
	# Context为自闭合标签，标签在在<Context />之间
	<Host name="localhost"  appBase="webapps"
        unpackWARs="true" autoDeploy="true">

	# ==================================增加内容	==================================
		<Context docBase="F:\test" path="/web"/>
	# ==================================增加内容	==================================	


        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    </Host>
	```

2. **访问地址。** 配置好项目路径后，如果要访问F:\test目录下的HelloWorld.html文件，只需要在浏览器中访问如下地址即可
  
	```bash
	# 以下地址中，前面部分和访问默认项目时的路径相同，只是在后面增加了自定义项目的
	# 虚拟路径名/web之后再接test目录中的html文件名即可
	http://localhost:8080/web/HelloWorld.html
	```
## 4.通过互联网访问本地项目
完成上述操作之后，就只剩下最关键的一个问题了：如何通过互联网访问本地内容。经过多番尝试，发现在同一个局域网内是可以通过`http://“对方的ip”:“对方的端口号”`访问到对方Tomcat的内容，但是一旦不在同一个局域网内就无法访问了。为了解决这个问题，笔者花了很大一部分时间查看了很多博客，但是都没有解决问题，最后结合几个博客的内容终于尝试成功，特此记录。

出现以上问题的原因在于，局域网中的主机可以互相通信，也可以访问局域网外的服务器。但是互联网的主机却没办法访问到局域网中的主机数据，因此需要内网穿透（或者叫映射）。即设置一个互联网可访问的ip地址，然后将其映射到内网的主机端口上，即可实现互联网的访问。为此，需要一个关键的软件：[花生壳5](https://hsk.oray.com/download/)。下载并注册该软件，即可获赠一个免费的域名。

得到域名之后需要实现该域名对本机地址的映射，具体操作惨见：[官方手册](http://service.oray.com/question/8146.html)

在此仅作简要说明：

1. **新增映射。** 运行并登录软件，点击下图所示的按钮，进入配置页面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331232554284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70)
2. **配置映射关系。** 如下图所示：1和2随意；3选择HTTP（刚开始无法选择，提示需要付费6元防止传播不良信息，这个逻辑实在无法理解。不过，老实交钱吧...）；4域名填写刚刚获得的免费域名，端口可以填80。因为80为HTTP的默认端口，这样一会儿访问的时候就不需要额外输入端口信息了；5内网主机直接填127.0.0.1，端口填你配置的Tomcat端口即可，默认为8080；6影响不大，随意即可。然后保存。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331233537566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MDUxMw==,size_16,color_FFFFFF,t_70)
3. 映射配置完成后，重新运行Tomcat服务器，这时就可以从互联网通过域名访问你主机的地址了，如上图所示，互联网可访问域名为xxx.iask.in，端口为80，则访问test目录下的HelloWorld.html文件的路径为：

	```bash
	http://xxx.iask.in/web/HelloWorld.html
	```
至此，通过互联网访问本地Tomcat项目的所有步骤就结束了，已经可以通过互联网轻松访问到你本地的项目了。你可以使用手机流量访问或者发送给朋友验证。
