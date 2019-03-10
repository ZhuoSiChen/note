### Enable 设计 作用是用来激活模块

WebMVC模块: @EnableWebMvc

Aspectj代理：@EnableAspectJAutoProxy

Caching 模块：@EnableCaching

Async 模块：@EnableAsync

与@Import

导入模块激活

例如: @EnableWebMvc

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({DelegatingWebMvcConfiguration.class})
public @interface EnableWebMvc {
}
```

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport{
    
}
```

所以导入了。

自定义实现的话

#### **Annotaition** 实现的话。

1. 实现 configuration 类

```
@Configuration
public class HelllordWorldConfiguration {
    @Bean
    public String helloWorld(){
		return "hello,World";
	}
}


```

2. 实现模块激活模式 annotation - `@EnableHelloWorld`

``` 
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({HelllordWorldConfiguration.class})
public @interface EnableHelloWrold {
}

```

3.标注@EnableHelloWorld 到引导类 EnableHelloWorldBootstrap

```java
@EnableHelloWorld
@Configuration
public class EnableHelloWorldBootstrap{
    public static void main(String[]args){
        AnnotaionConfigApplicationContext context = new  AnnotationConfigApplicationContext();
        //注册当前引导类（被@Configuration 标注）到Spring 上下文
        context.register(EnableHelloWorldBootStrap.class);
        // 启动上下文
        context.refresh();
        // 获取名称为 HelloWorld bean对象
        String helloWorld = context.getBean("helloWorld",String.class);
		System.out.println("helloWorld=%s \n",helloWorld);          
    }
}
```

#### **API **实现方式

需要实现ImportSelector 或 ImportBeanDefinitionRegistrar 接口。

 第一种 

* `ImportSelector `   参考` @EnableCaching`

  ​	先在构造器拿到注解`@EnableCircuitBreaker`，然后通过SpringFactoriesLoader.loader.loadFactoryNames(this.annotationClass, this.beanClassLoader); 拿到在 

  spring.fatories上定义的org.springframework.cloud.netflix.hystrix.HystrixCircuitBreakerConfiguration

  ```java
  @Configuration
  public class HystrixCircuitBreakerConfiguration {
      @Bean
      public HystrixCommandAspect hystrixCommandAspect() {
          return new HystrixCommandAspect();
      }
      @Bean
      public HystrixShutdownHook hystrixShutdownHook() {
          return new HystrixShutdownHook();
      }
  
      @Bean
      public HasFeatures hystrixFeature() {
          return HasFeatures.namedFeatures(new NamedFeature("Hystrix", HystrixCommandAspect.class));
          }
  }
  ```

  因为

  ```java
  protected SpringFactoryImportSelector() {
     this.annotationClass = (Class<T>) GenericTypeResolver
           .resolveTypeArgument(this.getClass(), SpringFactoryImportSelector.class);
  }
  
  // Find all possible auto configuration classes, filtering duplicates
  List<String> factories = new ArrayList<>(new LinkedHashSet<>(SpringFactoriesLoader
  				.loadFactoryNames(this.annotationClass, this.beanClassLoader)));
  ```

  然后加载 `HystrixCircuitBreakerConfiguration` 

  ```
  @Configuration
  public class HystrixCircuitBreakerConfiguration {
      @Bean
  	public HystrixCommandAspect hystrixCommandAspect() {
  		return new HystrixCommandAspect();
  	}
  
  	@Bean
  	public HystrixShutdownHook hystrixShutdownHook() {
  		return new HystrixShutdownHook();
  	}
  
  	@Bean
  	public HasFeatures hystrixFeature() {
  		return HasFeatures.namedFeatures(new NamedFeature("Hystrix", HystrixCommandAspect.class));
  	}
  
  }
  ```



​	然后在 `HystrixCommandAspect` 通过AspectJ AOP拦截注解了`@HystrixCommand` 的方法。

```java
static {//初始化工厂类
        META_HOLDER_FACTORY_MAP = ImmutableMap.<HystrixPointcutType, MetaHolderFactory>builder()
                .put(HystrixPointcutType.COMMAND, new CommandMetaHolderFactory())
                .put(HystrixPointcutType.COLLAPSER, new CollapserMetaHolderFactory())
                .build();
    }


@Aspect
public class HystrixCommandAspect {
 @Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand)")

    public void hystrixCommandAnnotationPointcut() {
    }

    @Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCollapser)")
    public void hystrixCollapserAnnotationPointcut() {
    }

    @Around("hystrixCommandAnnotationPointcut() || hystrixCollapserAnnotationPointcut()")
    public Object methodsAnnotatedWithHystrixCommand(final ProceedingJoinPoint joinPoint) throws Throwable {
        
         MetaHolderFactory metaHolderFactory = META_HOLDER_FACTORY_MAP.get(HystrixPointcutType.of(method));//拿到初始化好的工厂类 CollapserMetaHolderFactory 或者是 CommandMetaHolderFactory
        MetaHolder metaHolder = metaHolderFactory.create(joinPoint);//创建定义到注解的元信息
        HystrixInvokable invokable = HystrixCommandFactory.getInstance().create(metaHolder);
        ExecutionType executionType = metaHolder.isCollapserAnnotationPresent() ；
                metaHolder.getCollapserExecutionType() : metaHolder.getExecutionType();//拿到方法的执行方式 分别有 ASYNCHRONOUS, SYNCHRONOUS,OBSERVABLE;
        result = CommandExecutor.execute(invokable, executionType, metaHolder); 
||		 根据执行方式选择上下其中之一
        	castToExecutable(invokable, executionType).execute();//先把可调用的，转换成可执行的 再执行进入到HystrixCommand的execute()
				queue().
                	Future<R> delegate = toObservable().toBlocking().toFuture();//command转成一个RxJava可观察的Observable 返回一个Future对象，实际上使用Future的方式包装了 Observable （RxJava）有关
        		get();//调用Future的get() 方法
		result = executeObservable(invokable, executionType, metaHolder);
  
        
    }
}
```

Hystrix 动态化配置

```java
import com.netflix.config.ConfigurationManager;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import com.segumentfault.spring.cloud.lesson8.api.UserService;
import com.segumentfault.spring.cloud.lesson8.domain.User;
import org.apache.commons.configuration.AbstractConfiguration;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.Collection;
import java.util.Collections;
import java.util.Random;

/**
 * 用户服务提供方 Controller
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 * @since 0.0.1
 */
@RestController
public class UserServiceProviderController {

    @Autowired
    private UserService userService;

    private final static Random random = new Random();

    @PostMapping("/user/save")
    public boolean saveUser(@RequestBody User user) {
        return userService.saveUser(user);
    }

    @GetMapping("/change")
    public String changeTheHystrixCommandProperties(int timeOut){
        String commandKey = "getUsers"; // must fit to the used key
        String prefix = "hystrix.command." + commandKey + ".";
        AbstractConfiguration config = ConfigurationManager.getConfigInstance();
        System.out.println("timeOut="+timeOut);
        config.setProperty(prefix + "execution.isolation.thread.timeoutInMilliseconds", timeOut);
        return "OK";
    }
    /**
     * 获取所有用户列表
     *
     * @return
     */
    @HystrixCommand(
            commandProperties = { // Command 配置
                    // 设置操作时间为 100 毫秒
                    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "100")
            },
            commandKey="getUsers",
            fallbackMethod = "fallbackForGetUsers" // 设置 fallback 方法
    )
    @GetMapping("/user/list")
    public Collection<User> getUsers() throws InterruptedException {

        long executeTime = random.nextInt(200);

        // 通过休眠来模拟执行时间
        System.out.println("Execute Time : " + executeTime + " ms");

        Thread.sleep(executeTime);

        return userService.findAll();
    }

    /**
     * {@link #getUsers()} 的 fallback 方法
     *
     * @return 空集合
     */
    public Collection<User> fallbackForGetUsers() {
        return Collections.emptyList();
    }

}

```

http://localhost:9090/archaius

可以获得更改后的配置

疑问：1.Hystrix 如何在调用时候从ConfigurationManager获取到属于当前的timeout值。



*** 以下是草稿

```java
MetaHolder metaHolder = metaHolderFactory.create(joinPoint);CommandMetaHolderFactory.create
public MetaHolder create(Object proxy, Method method, Object obj, Object[] args, final ProceedingJoinPoint joinPoint)
HystrixCommand hystrixCommand = method.getAnnotation(HystrixCommand.class);//获取hystrixCommand 的相应配置。
```

也就是在`HystrixCommandProperties`这个类中的所有配置

``` java
MetaHolder.Builder builder = metaHolderBuilder(proxy, method, obj, args, joinPoint);
然后根据配置项里的在通过metaHolderBuilder转换配置项的实际意思 
然后在根据
hystrixCommand.observableExecutionMode()
的语义创建出相应的metaHolder(元信息持有者)
```

```java
methodsAnnotatedWithHystrixCommand(final ProceedingJoinPoint joinPoint)
	MetaHolderFactory metaHolderFactory = META_HOLDER_FACTORY_MAP.get(HystrixPointcutType.of(method));//拿到静态构建的CommandMetaHolderFactory
	MetaHolder metaHolder = metaHolderFactory.create(joinPoint);//根据方法拿到，创建HystrixCommand元信息，也就是配置在HystrixCommand注解中的信息。
		CommandMetaHolderFactory.create(Object proxy, Method method, Object obj, Object[] args, final ProceedingJoinPoint joinPoint) 
		HystrixCommand hystrixCommand = method.getAnnotation(HystrixCommand.class);//获取注解，根据注解构造元信息。
	HystrixInvokable invokable = HystrixCommandFactory.getInstance().create(metaHolder);//根据元信息，构造Command
		 executable = new CommandCollapser(metaHolder);
||
		 executable = new GenericObservableCommand(HystrixCommandBuilderFactory.getInstance().create(metaHolder));
||
		 executable = new GenericCommand(HystrixCommandBuilderFactory.getInstance().create(metaHolder))//这个方法会构建的是 AbstractHystrixCommand<T>的实例，所以可以通过继承这个类的方法构建HystrixCommand的实现
			*HystrixCommandBuilderFactory.getInstance()//拿到工厂实例创建对象				
            create(metaHolder);//根据元信息创建command对象
				create(metaHolder, Collections.<HystrixCollapser.CollapsedRequest<Object, Object>>emptyList());
					validateMetaHolder(metaHolder);//校验元信息合不合法
					return HystrixCommandBuilder.builder()//拿到建造者
                .setterBuilder(createGenericSetterBuilder(metaHolder))//创建通用的Setter
                .commandActions(createCommandActions(metaHolder))
                .collapsedRequests(collapsedRequests)
                .cacheResultInvocationContext(createCacheResultInvocationContext(metaHolder))
                .cacheRemoveInvocationContext(createCacheRemoveInvocationContext(metaHolder))
                .ignoreExceptions(metaHolder.getCommandIgnoreExceptions())
                .executionType(metaHolder.getExecutionType())
                .build();
               
			
	result = CommandExecutor.execute(invokable, executionType, metaHolder);
		castToExecutable(invokable, executionType).execute();//先把可调用的，转换成可执行的在执行进入到HystrixCommand的execute()
			queue().
                final Future<R> delegate = toObservable().toBlocking().toFuture();//command转成一个RxJava可观察的Observable 返回一个Future对象，
            get();

			
||
    result = executeObservable(invokable, executionType, metaHolder);


```



​	



* `ImportBeanDefinitionRegistrar`