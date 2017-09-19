---
title: WebServlet+Jetty+Nginx轻松开发并部署应用服务器
tags:
- 后端
- Jetty
- Nginx
- WebServlet
- Idea
categories: 后端
thumbnail: https://ooo.0o0.ooo/2017/06/14/5940bc236ca73.png
---

本文教你如何使用 WebServlet + Jetty + Nginx 从开发到部署应用服务器！

# 由来

我最近需要为 Dir 开发后端应用服务器，以前是用的某SaaS服务平台（也是WebServlet），觉得太贵，于是需要部署到自己的服务器。目前有比较成熟的 Python 、PHP 和 JavaScript等语言 可用，但是我需要尽快开发出来，不能花时间新学一门语言，于是就选择了一直在用的 WebServlet。

# 开始开发

我默认你已会Java，对WebServlet有了解。关于WebServlet，可以参考：[http://www.runoob.com/servlet](http://www.runoob.com/servlet) 和 [http://www.importnew.com/14621.html](http://www.importnew.com/14621.html) 。

本文使用 Intellij Idea 社区版 作为开发和调试环境。

## 搭建开发环境

IntelliJ Idea 不资瓷Web开发，所以我们需要手动写一些配置。参考：[http://qinghua.github.io/intellij-idea-community-hotswap-webapp/](http://qinghua.github.io/intellij-idea-community-hotswap-webapp/) 。

* 首先，新建一个项目。

  * 打开Idea，点击 `Create new project` 新建项目。左侧选择 `Maven`，Project SDK选 `1.8`。

    ![选择Maven](https://ooo.0o0.ooo/2017/06/11/593d2920f3f94.png)


  * 然后Next，输入你的 `GroupId`、`ArtifactId` 以及 `Version`。![输入信息](https://ooo.0o0.ooo/2017/06/11/593d29ce6a7b2.png)

  Next，输入路径，最后Finish。

## 配置项目

* 首先进入项目，打开 `pom.xml`。

  ![我这里pom.xml](https://ooo.0o0.ooo/2017/06/11/593d2a2754821.png)

* 在 `project` 内新建 `build`，填写下列XML代码：

  ```xml
  <plugins>
        <plugin>
          <groupId>org.mortbay.jetty</groupId>
          <artifactId>maven-jetty-plugin</artifactId>
          <version>6.1.26</version>
          <configuration>
            <contextPath>/</contextPath>
            <scanIntervalSeconds>0</scanIntervalSeconds>
            <connectors>
              <connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
                <port>8080</port>
                <maxIdleTime>60000</maxIdleTime>
              </connector>
            </connectors>
          </configuration>
        </plugin>
      </plugins>
  ```

* 然后在菜单 `Run` -> `Edit Configurations` 里，新建一个`Maven ` 根节点（直接点绿色加号就行），然后填入如图参数：

  ![填写参数](https://ooo.0o0.ooo/2017/06/11/593d2c154f103.png)

  * Name：Jetty
  * Working directory：默认就好
  * Command line：jetty:run

* 然后选择 `Runner` 标签，把 `Use Project Settings` 去掉，填入参数：

  ![Runner标签填入参数](https://ooo.0o0.ooo/2017/06/11/593d2fb557d1e.png)

  * VM Options：-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=6006

    端口6006可以自行决定，别和其它端口冲突就行。

* 再点击绿色+号，新建一个根节点： `Remote`，填入参数：

  ![Remote](https://ooo.0o0.ooo/2017/06/11/593d30650c638.png)

  * Name：Jetty-Remote-Debug
  * Port：6006 （和先前在`Runner`标签里填写的端口一致就行。）

  随后点击 OK 退出。

* 再次回到 `pom.xml`，添加 `Jetty` 依赖：

  ```xml
      <dependencies>
          <dependency>
              <groupId>org.eclipse.jetty</groupId>
              <artifactId>jetty-server</artifactId>
              <version>9.4.3.v20170317</version>
          </dependency>
      </dependencies>
  ```

  然后按 `Ctrl+S` 保存，在右下角弹出通知中点击 `Import Changs`，等一会儿加载完成。

## 开始开发

* 在左侧 Project 窗格打开 `src -> main -> java` 照常新建包和Servlett，可以参照上面给出的两个教程，不多阐述：

```java
public class HelloWorldServlet extends HttpServlet {
    @Override
    public void doGet (HttpServletRequest request,
                       HttpServletResponse response) throws ServletException,
            IOException {
        // 设置响应内容类型
        response.setContentType("text/html");

        // 实际的逻辑是在这里
        PrintWriter out = response.getWriter();
        out.println("Hello World");
    }
}
```
* 配置Servlet

* * 左侧 Project 窗格选择 Main，点击右键新建目录，名叫 `webapp`，里面新建 `WEB-INF` 目录，创建 `web.xml` 配置文件：（会提示URL is not registered，不用管它）

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    </web-app>
    ```

  * 在里面加入你的 Servlet:

    ```xml
    	<servlet>
    		<servlet-name>HelloWorldServlet</servlet-name>
    		<servlet-class>top.trumeet.demo.HelloWorldServlet</servlet-class>
    	</servlet>
    	<servlet-mapping>
    		<servlet-name>HelloWorldServlet</servlet-name>
    		<url-pattern>/welcome</url-pattern>
    	</servlet-mapping>
    ```

## 运行

* 运行很简单，执行 `Jetty` 运行即可：

![运行程序](https://ooo.0o0.ooo/2017/06/11/593d345579da6.png)

* 等待日志出现 `[INFO] Started Jetty Server` 即成功了。

  ![日志](https://ooo.0o0.ooo/2017/06/11/593d3455e4226.png)

* 这时，访问 [http://127.0.0.1:8080/welcome](http://127.0.0.1:8080/welcome) （8080是你pom.xml中plugin里设定的端口）即可看到 `Hello World` 辣！

  ![HelloWorld](https://ooo.0o0.ooo/2017/06/11/593d34fce93e1.png)



# 部署到服务器

## 在服务器上安装Jetty

* 安装JDK1.8

  > 参考：[http://www.yiibai.com/article/4439.html](http://www.yiibai.com/article/4439.html)

  （比较简单，就不Copy了 XD）

* 安装Jetty

  * 从官网下载，打开官网 [http://www.eclipse.org/jetty/](http://www.eclipse.org/jetty/)，找到最新下载直链，进行下载：

    ```shell
    wget http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/9.4.6.v20170531/jetty-distribution-9.4.6.v20170531.tar.gz
    ```

  * 解压：

    ```shell
    tar -xzvf jetty-distribution-9.4.6.v20170531.tar.gz
    ```

## 编译war

到AS右面 `Maven Projects` 窗格 选择 `Lifecycle -> package` 编译。

![](https://ooo.0o0.ooo/2017/06/12/593dd818ef5aa.png)

双击启动，等待日志出现 `Process finished with exit code 0` 即表示编译完成（一般几秒钟就好吧..）

在上面 `[INFO] Building war: XXX.war` 处找到war位置。

## 部署war

* 启动服务器

  我们使用 jetty.sh start 启动服务器。

  ```shell
  jetty-distribution-9.4.6.v20170531/bin/jetty.sh start
  ```

  等待 `OK` 之后即可访问 `服务器IP:8080` 就会出现无已部署应用的404页面，即表示安装Jetty成功。

* 上传到服务器

  有两种方式部署war，第一种是直接把war放到 `jetty/webapps` 下，然后通过 `<Jetty监听地址>/XXX.war` 访问。另一种是放到任何可以访问的地方，然后在 `jetty/webapps` 新建一个XML用于访问。

  - 第一种方案

    很简单，把war用SFTP等方式上传到 `webapps` 下即可。如 `jetty/webapps/Demo.war` ，就需要访问 `<你的IP>:8080/Demo` 。

  - 第二种方案

    首先，把war放到你需要的目录，如 `~` ，名称随意。然后在 `jetty/webapps` 新建 `Demo.xml`，填入以下内容：

    ```xml
    <?xml version="1.0"  encoding="UTF-8"?>
    <!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">

    <Configure class="org.eclipse.jetty.webapp.WebAppContext">

      <Set name="contextPath">/demo</Set>
      <Set name="war">/root/XXX.war</Set>

    </Configure>
    ```

    里面 `contextPath` 就是 `<你的IP>:8080/` 后面的地址。`war` 是war的 **绝对路径**。

* 重启Jetty

  每次更新war都需要重启jetty，使用 `jetty.sh restart` 即可。


  * 验证效果

    至此，我们已经部署好了你的应用，现在打开 `<你的服务器IP>:8080/demo/welcome` 即可访问到刚才的 HelloWorld 了。（说一下，8080是端口号，我这里一直保持的Jetty默认，需要修改端口自行Google）


## Nginx

我们希望通过诸如 `api.example.com` 访问到我们的应用，所以需要配置Nginx。

我这里用的lnmp一键安装包，所以这样做：

* 新建一个vHost

  `lnmp vhost add`

  域名自己填，如 `api.example.com`，剩下的.. 自己填（X

* 修改Nginx配置文件

  打开 `/usr/local/nginx/conf/vhost/<刚才填的第一个域名>.conf` （不是lnmp可能不在这里，大家自己找一下）

  在 `server` 块内追加如下内容：

  ```nginx
  location / {
           proxy_pass      http://127.0.0.1:8080;
           proxy_redirect  off;
           proxy_set_header        Host            $host;
           proxy_set_header        X-Real-IP       $remote_addr;
           proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
     }
  ```

  **这里的 8080 就是刚才Jetty设定的端口，如果你修改了，这里也需要同时修改。**

  然后保存退出。最后 `lnmp nginx restart` 重启nginx。

* 添加域名解析

  这个自己去域名服务商或者CloudFlare之类的服务商填就行。

最终，我们可以通过 `api.example.com/demo/welcome` 看到 `Hello World` 了。

​

# 更新Webapp

（这方面没有仔细研究，轻拍）

如果我们的程序做了修改，我的做法是重新打包，上传，最后重启Jetty即可。

-----

还要说一下，我没有设定Jetty开机自启，所以每次重启服务器都需要 `jetty.sh start` 启动一下。

> 本博客均为原创内容，未经允许不得转载！