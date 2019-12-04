# 一、pom文件解析
1. spring-boot-starter-parent：继承spring-boot-starter-parent包来获得一些合理的默认配置

2. spring-boot-starter：Spring Boot的核心启动器，包含了自动配置、日志和YAML

3. spring-boot-starter-web：将项目变成web项目，包含RESTful风格框架SpringMVC和默认的嵌入式容器Tomcat，本身也依赖spring-boot-starter

4. mybatis-spring-boot-starter：spring boot和mybatis的整合包

# 二、@SpringBootApplication
1. 启动springboot项目的注解，替代了@@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan这三个注解的功能

2. @SpringBootConfiguration，继承自@Configuration，一般与@Bean配合使用，使用这两个注解就可以创建一个简单的spring配置类，可以用来**替代相应的xml配置文件**

3. @EnableAutoConfiguration，可以根据你添加的jar包来配置你项目的默认配置，比如当你添加了mvc的jar包，它就会自动配置web项目所需的配置

4. @ComponentScan：扫描组件，只要组件上有@component及其子注解@Service、@Repository、@Controller等，springboot会自动扫描到并纳入Spring 容器进行管理，有点类似xml文件中的<context:component-scan>，该注解不填属性的话就是默认扫描启动类所在的包，或者启动类所在包的下一级，所以**启动类要放在最外层**

# 三、通过配置类和注解代替配置文件
1. springboot可以把以前写的配置文件的内容都写进Config配置类里面，并且在配置类打上Configuration注解，告诉spring这是个配置类，扫描这个类里的配置信息

   1. 在类里面的配置属性上面加@Value注解引入properties文件的值
      ```
             @Value("${spring.datasource.url}")
             private String url;
      ```
   
   2. 在类里面用@Bean注解定义一个方法返回一个类被spring管理
      1. 类的属性可以在方法里面定义
         ```
            @Bean("productConfig")
            public ProductConfig productConfig(){
                ProductConfig pc = new ProductConfig();
                pc.age = 11;
                pc.name = "gunners";
                return pc;
            }
         ```
      
      2. 也可以用@ConfigurationProperties注解引入properties文件的值，但是要是yml文件的格式
         ```
            @Bean
            @ConfigurationProperties(prefix = "spring.datasource") //这里要引用yml文件的数据
            public DataSource dataSource(){
                return new DruidDataSource();
            }
         ```

3. 如果实体类在属性上面加@Value注解引入properties文件的值的同时别的类中也用@Bean注解定义了该实体类并给属性赋值（就是上面1,2种情况），那么实体类自动注入后属性的默认值还是@Value注解引入的properties定义的值

# 四、IDEA+springboot静态资源访问的坑

   1. 在SpringBoot中，默认配置的/** 映射到/static，所以静态资源文件（例如html、css等）要放在resources下的static文件夹，然后引用的时候直接写:/xx.html就好，不用/static/xx.html，否则会报404


   
   






