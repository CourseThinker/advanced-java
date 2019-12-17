**SpringBoot启动过程分析**

1.创建一个SpringApplication对象，执行构造方法；

2.调用初始化方法：initialize

	1.将source放入SpringApplication的sources属性中管理，sources是一个LinkedHashSet()，这意味着我们可以同时创建多个自定义不重复的Application，但是目前只有一个。
```
2.判断是否是web程序(javax.servlet.Servlet和org.springframework.web.context.ConfigurableWebApplicationContext都必须在类加载器中存在)，并设置到webEnvironment属性中 。
```

```
3.从spring.factories中找出ApplicationContextInitializer并设置到初始化器initializers。
```

```
4.从spring.factories中找出ApplicationListener，并实例化后设置到SpringApplication的监听器listeners属性中。这个过程就是找出所有的应用程序事件监听器。
```

```
5.找出的main方法的类（这里是CmsApplication），并返回Class对象。
```

3.执行run方法

```
1.创建了应用的监听器SpringApplicationRunListeners并开始监听；
```

```
2.加载SpringBoot配置环境(ConfigurableEnvironment)，如果是通过web容器发布，会加载StandardEnvironment，其最终也是继承了ConfigurableEnvironment；
```

```
3.配置环境(Environment)加入到监听器对象中(SpringApplicationRunListeners)；
```

```
4.创建Spring容器：ConfigurableApplicationContext(应用配置上下文)；
```

```
5.回到run方法内，设置容器prepareContext方法，将listeners、environment、applicationArguments、banner等重要组件与上下文对象关联；
```

```
6.刷新容器，refresh()方法；
```

```
7.广播出ApplicationReadyEvent，执行结束返回ConfigurableApplicationContext；
```

