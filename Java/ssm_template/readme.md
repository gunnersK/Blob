# 这是一个ssm的模板

1. 如果用eclipse导入的话。会出现一个在这里看不到的parent聚合父项目
2. 当前目录下的pom文件就是parent的pom文件
3. 放在WEB-INF下的文件都无法直接从地址栏访问到，只有配置控制器，通过虚拟路径进图控制器跳转
4. 关于视图解析器(加前后缀)，一般前缀是/WEB-INF/jsp，也不一定是要jsp目录，叫啥都行后缀是.jsp。这样你在控制器跳转的时候，return一个文件名就行了，/WEB-INF/jsp前缀和.jsp后缀都会帮你自动加
5. 如果想用html写页面，那应该放在WEB-INF里面还是外面呢？
   1. 如果放在WEB-INF中，视图解析器的后缀配.html是没用的，因为他只对.jsp才起作用，应该是这样，反正配.html肯定没用
   2. 如果把html挪到WEB-INF外面，即项目根目录下，这样就能随便访问了，但是因为你要用控制器，所以在web.xml里面要开springmvc的拦截，我们是定义拦截所有，即（/），这样是除了jsp文件，其他请求都会被拦截下来，走springmvc.xml里面定义的规则，这样除了jsp，其他都不能直接访问了，这就是为什么要在springmvc.xml里面定义对css/js等静态资源的放行，这样html又访问不了，就只能跟css/js一样定义放行，多定义一条规则
6. 各个项目pom文件需要写的内容
   1. parent--资源文件拷贝插件、java编译插件、Tomcat插件（定义端口啥的）
   2. web--servlet、文件上传
   3. service--spring（需要web依赖service，因为web层也要用spring）
   4. dao--数据库相关，分页啥的
