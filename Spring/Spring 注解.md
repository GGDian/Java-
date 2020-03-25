### @ComponentScan

```java
// 可以包含多个 ComponentScan
@ComponentScans(   
      value = {
            @ComponentScan(value="com.atguigu",includeFilters = {
                // 按照注解
/*                @Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
                  @Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/ // 指定的类
                
                // 自定义规则
                  @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
            },useDefaultFilters = false)   // 需要把默认的包扫描策略置为 false
      }
      )
```

```java
//@ComponentScan  value:指定要扫描的包
//excludeFilters = 按Filter[] ：指定扫描的时候照什么规则排除那些组件
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型；
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则
```

```java
public class MyTypeFilter implements TypeFilter {
   /**
    * metadataReader：读取到的当前正在扫描的类的信息
    * metadataReaderFactory:可以获取到其他任何类信息的
    */
   @Override
   public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
         throws IOException {
      // TODO Auto-generated method stub
      //获取当前类注解的信息
      AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
      //获取当前正在扫描的类的类信息
      ClassMetadata classMetadata = metadataReader.getClassMetadata();
      //获取当前类资源（类的路径）
      Resource resource = metadataReader.getResource();
      
      String className = classMetadata.getClassName();
      System.out.println("--->"+className);
      if(className.contains("er")){
         return true;
      }
      return false;
   }
}
```

### @Conditional

可以标在类上，也可以标在方法上

```java
@Conditional({Condition}) ： 按照一定的条件进行判断，满足条件给容器中注册bean
```

condition 条件返回 true 才会创建 bean 



```java
@Target({ElementType.TYPE, ElementType.METHOD})
```

```java
	@Conditional(LinuxCondition.class)
	@Bean("linus")
	public Person person02(){
		return new Person("linus", 48);
	}
```

```java
//判断是否linux系统
public class LinuxCondition implements Condition {
   /**
    * ConditionContext：判断条件能使用的上下文（环境）
    * AnnotatedTypeMetadata：注释信息
    */
   @Override
   public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
      // TODO是否linux系统
      //1、能获取到ioc使用的beanfactory
      ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
      //2、获取类加载器
      ClassLoader classLoader = context.getClassLoader();
      //3、获取当前环境信息
      Environment environment = context.getEnvironment();
      //4、获取到bean定义的注册类
      BeanDefinitionRegistry registry = context.getRegistry();
      
      String property = environment.getProperty("os.name");
      
      //可以判断容器中的bean注册情况，也可以给容器中注册bean
      boolean definition = registry.containsBeanDefinition("person");
      if(property.contains("linux")){
         return true;
      }
      return false;
   }
}
```

### @Import

```
* 3）、@Import[快速给容器中导入一个组件]
*     1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
*     2）、ImportSelector:返回需要导入的组件的全类名数组；
*     3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
* 4）、使用Spring提供的 FactoryBean（工厂Bean）;
*     1）、默认获取到的是工厂bean调用getObject创建的对象
*     2）、要获取工厂Bean本身，我们需要给id前面加一个&
*        &colorFactoryBean
```

ImportSelector

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
	//返回值，就是到导入到容器中的组件全类名
	//AnnotationMetadata:当前标注@Import注解的类的所有注解信息
	@Override
	public String[] selectImports(AnnotationMetadata importingClassMetadata) {
		// TODO Auto-generated method stub
		//importingClassMetadata
		//方法不要返回null值
		return new String[]{"com.atguigu.bean.Blue","com.atguigu.bean.Yellow"};
	}
}
```

ImportBeanDefinitionRegistrar

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

   /**
    * AnnotationMetadata：当前类的注解信息
    * BeanDefinitionRegistry:BeanDefinition注册类；
    *        把所有需要添加到容器中的bean；调用
    *        BeanDefinitionRegistry.registerBeanDefinition手工注册进来
    */
   @Override
   public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      
      boolean definition = registry.containsBeanDefinition("com.atguigu.bean.Red");
      boolean definition2 = registry.containsBeanDefinition("com.atguigu.bean.Blue");
      if(definition && definition2){
         //指定Bean定义信息；（Bean的类型，Bean。。。）
         RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
         //注册一个Bean，指定bean名
         registry.registerBeanDefinition("rainBow", beanDefinition);
      }
   }
```



### FactoryBean

* 使用Spring提供的 FactoryBean（工厂Bean）;
  * 默认获取到的是工厂bean调用getObject创建的对象
  * 要获取工厂Bean本身，我们需要给id前面加一个&
* 即容器中有两个对象
* ![image-20200321150850773](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200321150850773.png)

```
public class ColorFactoryBean implements FactoryBean<Color> 
```

```
@Bean
public ColorFactoryBean colorFactoryBean() {
    return new ColorFactoryBean();
}
```

 

### Bean 的生命周期

```
销毁：
     单实例：容器关闭的时候
     多实例：容器不会管理这个bean；容器不会调用销毁方法；
```



### @Value

```
//使用@Value赋值；
//1、基本数值
//2、可以写SpEL； #{}
//3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）
```



### Autowired

 1）、@Autowired：自动注入：
  		1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
  		2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
  							applicationContext.getBean("bookDao")
  		3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
  		4）、自动装配默认一定要将属性赋值好，没有就会报错；
  			可以使用@Autowired(required=false);
  		5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
  				也可以继续使用@Qualifier指定需要装配的bean的名字

2）、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
  		@Resource:
  			可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；
  			没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
  		@Inject:
  			需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
   @Autowired:Spring定义的； @Resource、@Inject都是java规范
  	
  AutowiredAnnotationBeanPostProcessor:解析完成自动装配功能；		

  3）、 @Autowired:构造器，参数，方法，属性；都是从容器中获取参数组件的值
  		1）、[标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配

```
//@Autowired 
//标注在方法，Spring容器创建当前对象，就会调用方法，完成赋值；
//方法使用的参数，自定义类型的值从ioc容器中获取
public void setCar(Car car) {
   this.car = car;
}
```

  		2）、[标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
  		3）、放在参数位置：

  4）、自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
  		自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
  		把Spring底层一些组件注入到自定义的Bean中；
  		xxxAware：功能使用xxxProcessor；
  			ApplicationContextAware==》ApplicationContextAwareProcessor；
  	