### Servlet

```javascript
Shared libraries（共享库） / runtimes pluggability（运行时插件能力）

1、Servlet容器启动会扫描，当前应用里面每一个jar包的
   ServletContainerInitializer的实现
2、提供ServletContainerInitializer的实现类；
   必须绑定在，META-INF/services/javax.servlet.ServletContainerInitializer
   文件的内容就是ServletContainerInitializer实现类的全类名；

总结：容器在启动应用的时候，会扫描当前应用每一个jar包里面
META-INF/services/javax.servlet.ServletContainerInitializer
指定的实现类，启动并运行这个实现类的方法；传入感兴趣的类型；


ServletContainerInitializer；
@HandlesTypes；
```





```java
//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类，子接口等）传递过来；
//传入感兴趣的类型；
@HandlesTypes(value={HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {

   /**
    * 应用启动的时候，会运行onStartup方法；
    * 
    * Set<Class<?>> arg0：感兴趣的类型的所有子类型；
    * ServletContext arg1:代表当前Web应用的ServletContext；一个Web应用一个ServletContext；
    * 
    * 1）、使用ServletContext注册Web组件（Servlet、Filter、Listener）
    * 2）、使用编码的方式，在项目启动的时候给ServletContext里面添加组件；
    *        必须在项目启动的时候来添加； (启动的时候啊啊啊啊啊)
    *        1）、ServletContainerInitializer得到的ServletContext；
    *        2）、ServletContextListener得到的ServletContext；（自己定义的监听器）
    */
   @Override
   public void onStartup(Set<Class<?>> arg0, ServletContext sc) throws ServletException {
      // TODO Auto-generated method stub
      System.out.println("感兴趣的类型：");
      for (Class<?> claz : arg0) {
         System.out.println(claz);
      }
      
      //注册组件  ServletRegistration  
      ServletRegistration.Dynamic servlet = sc.addServlet("userServlet", new UserServlet());
      //配置servlet的映射信息
      servlet.addMapping("/user");
      
      
      //注册Listener
      sc.addListener(UserListener.class);
      
      //注册Filter  FilterRegistration
      FilterRegistration.Dynamic filter = sc.addFilter("userFilter", UserFilter.class);
      //配置Filter的映射信息
      filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");
      
   }

}
```





### Spring MVC

```java
1、web容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer
2、加载这个文件指定的类SpringServletContainerInitializer
3、spring的应用一启动会加载感兴趣的WebApplicationInitializer接口的下的所有组件；
4、并且为WebApplicationInitializer组件创建对象（组件不是接口，不是抽象类）
   1）、AbstractContextLoaderInitializer：创建根容器；createRootApplicationContext()；
   2）、AbstractDispatcherServletInitializer：
         创建一个web的ioc容器；createServletApplicationContext();
         创建了DispatcherServlet；createDispatcherServlet()；
         将创建的DispatcherServlet添加到ServletContext中；
            getServletMappings();
   3）、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
         创建根容器：createRootApplicationContext()
               getRootConfigClasses();传入一个配置类
         创建web的ioc容器： createServletApplicationContext();
               获取配置类；getServletConfigClasses();
   
总结：
   以注解方式来启动SpringMVC；继承AbstractAnnotationConfigDispatcherServletInitializer；
实现抽象方法指定DispatcherServlet的配置信息；

===========================
定制SpringMVC；
1）、@EnableWebMvc:开启SpringMVC定制配置功能；
   <mvc:annotation-driven/>；

2）、配置组件（视图解析器、视图映射、静态资源映射、拦截器。。。）
   extends WebMvcConfigurerAdapter

```



![image-20200218184625481](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200218184625481.png)

![image-20200218184648518](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200218184648518.png)





上下文切换：

多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。