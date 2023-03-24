# 1. 创建tomcat开发环境

1. Run --> Edit Configurations --> Templates --> Tomcat Server --> Local(本地)/Remote(远程)

2. Configure --> 配置Tomcat --> Ok

3. New Module --> Java Enterprise --> Java EE 7 --> Web Application --> Version 3.1 --> Next --> ... --> Finish

# 2. Servlet 2.5

1. 创建JavaEE项目, 选择Servlet的版本3.0以上. 可以不创建Web.xml;

2. 定义一个类, 实现Servlet接口;

3. Overwrite方法

    ```
    public class Demo01_Servlet implements Servlet {

    /**
     * 初始化方法
     * 在Servlet被创建时执行, 只会执行一次, 一般用来加载资源
     * 默认情况下, 第一次被访问时, Servlet被创建
     * 在web.xml <servlet>标签下修改
     * Servlet的init方法只执行一次, 索命一个Servlet在内存中只存在以一个对象, Servlet是单例的
     * 多个用户同时访问时, 可能存在线程安全问题;   解决: 尽量不一样要在Servlet中定义成员变量, 即使定义的,也不要对其修改值;
     *
     * @param servletConfig
     * @throws ServletException
     */
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("init.....");

    }

    /**
     * 获取ServletConfig对象
     * ServletConfig: Servlet的配置对象
     * @return
     */
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    /**
     * 提供服务的方法
     * 每一次Servlet被访问时执行, 执行多次
     * @param servletRequest
     * @param servletResponse
     * @throws ServletException
     * @throws IOException
     */
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("service.....");
    }

    /**
     * 获取Servlet的一些信息, 版本 作者...
     * @return
     */
    @Override
    public String getServletInfo() {
        return null;
    }

    /**
     * 销毁方法
     * 在Servlet被杀死时执行 or Tomcat服务正常关闭时执行, 执行一次
     * 在Servlet被销毁前执行, 一般用来释放资源;
     */
    @Override
    public void destroy() {
        System.out.println("destory.....");
    }
}
    ```

4. 配置web.xml

    ```
    <servlet>
        <servlet-name>Demo01_Servlet</servlet-name>
        <servlet-class>com.alec.web.servlet.Demo01_Servlet</servlet-class>
    
        <!-- 指定Servlet创建时机
            1. 第一次被访问时, 创建
                <load-on-startup> 为负数, 默认-1
            2. 在服务器启动时, 创建
                <load-on-startup> 为自然数, 一般0-10
         -->
        <load-on-startup>5</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>Demo01_Servlet</servlet-name>
        <url-pattern>/demo01</url-pattern>
    </servlet-mapping>
    ```
    
# 3. Servlet 3.0
1. 创建JavaEE项目, 选择Servlet的版本3.0以上. 可以不创建Web.xml;

2. 定义一个类, 实现Servlet接口;

3. Overwrite方法;

4. 在类上使用@WebServlet注解, 进行配置;

    ```
    @WebServlet(urlPatterns = "/demo01")
    or
    @WebServlet("/demo01")
    ```
## 3.1 Servlet体系结构
- Servlet 接口
  - GenericServlet 抽象类: 
    - HttpServlet 抽象类

### 3.1.1 GenericServlet

- 将Servlet接口中的其他方法做了默认空实现, 只将service()方法做了abstract;
- 使用时, 只需要实现service()方法即可;
- **一般不用**;

### 3.1.2 HttpServlet

- 对Http协议的封装, 可以简化操作
- 定义类 继承 HttpServlet
- 复写doGet/doPost方法

## 3.2 Servlet 配置

 - urlpartten
   - 一个Servlet可以定义多个访问路径: `@WebServlet({"/d4", "/demo04", "/demo_servlet4"})`
   - 路径定义规则:
     - /xxx
     - /xxx/xxx : 多层路径, 目录结构
     - *.do    : http://localhost:8080/aaa.do or http://localhost:8080/aaa/bbb.do


# 4 Request

## 4.1 服务器响应浏览器请求流程

1. tomcat服务器会根据请求url中的资源路径, 创建对应的ServletDemo的对象;
2. tomcat服务器, 会创建reqquest和response对象, request对象中封装请求消息数据;
3. tomcat将request和response两个对象传递给service方法, 并调用service方法;
4. 程序员, 可以通过request对象获取请求消息数据, 通过response对象设置相应消息数据;
5. 服务器在给浏览器做出相应之前, 会从response对象中拿程序员设置的响应消息数据;

## 4.2 Request对象
- Reqquest对象: 获取请求消息;
- Response对象: 设置响应消息;

> Request对象继承关系 ServletRequest(接口) --> HttpServletRequest(接口) --> org.apache.catalina.connector.RequestFacade(tomcat实现类)

## 4.3 Request功能
```
1. 获取请求的消息数据
1) 获取请求行数据
    GET /c02/demo01?name=alec HTTP/1.1
    方法:
        1> 获取请求方式: GET
            String getMethod()
        2> 获取虚拟目录: /
            String getContextPath()
        3> 获取Servlet路径: /c02/demo01
            String getServletPath()
        4> 获取get方式请求参数
            String getQueryString()
        5> 获取请求URI: /c02/demo01
            String getRequestURI(): /c02/demo01
            StringBuffer getRequestURL(): http://localhost:8080/c02/demo01
                URI: 统一资源识别符
                URL: 统一资源定位符
        6> 获取协议及版本: HTTP/1.1
            String getProtocol()
        7> 获取客户机的IP地址:
            String getRemoteAddr()
            
  2) 获取请求头数据
    方法:
        String getheader(String name): 通过请求头的名称获取请求头的值
        Enumeration<String> getHeaderNames(): 获取所有请求头名称
            Enumeration枚举接口, 可以当做迭代器使用;
            
  3) 获取请求体数据
    请求体: 只有POST请求方式, 才有请求体, 在请求体重封装了POST请求的请求参数
    步骤:
        1> 获取流对象
            BufferedReader getReader()  获取字符输入流
            ServletInputStream getInputStream() 获取字节输入流
                在文件上传知识点讲解
        2> 再从流对象中拿数据
        
2. 其他功能
  1) 获取请求参数通用方式(不论POST还是GET都可以使用下面的方法)
    1> String getParameter(String name): 根据参数名称获取参数值      username=alec&password=admin
    2> String[] getParameterValues(String name): 根据参数名称获取参数值的数组   hobby=music&hobby=eat&hobby=game
    3> Enumeration<String> getParameterNames(): 获取所有请求参数的参数名称
    4> Map<String, String[]> getParameterMap(): 获取所有请求参数的Map集合
    
    中文乱码问题:
        get方式: tomcat > 8  已经将get方式的乱码解决
        post方式: 设置流的编码: request.setCharacterEncoding("utf-8");
    
  2) 请求转发: 一种在服务器内部的资源跳转方式
    1> 步骤:
        1>> 通过request对象获取请求转发器对象: RequestDispatcher getRequestDispatcher(String path)
        2>> 使用RequestDispatcher对象进行转发: forward(ServletReuqest request, ServletRespose response)
    2> 特点:
        1>> 浏览器地址栏不发生变化;
        2>> 只能访问当前服务器内部资源, 不能访问外部url;
        3>> 转发是一次请求;
  
  3) 数据共享
    1> 域对象: 一个有作用范围的对象, 可以在范围内共享数据
    
  
  4) 获取ServletContext
```