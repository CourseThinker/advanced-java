**SpringBean的生命周期**

**1.实例化：**

实例化一个Bean，即创建一个对象，常说的new；

**2.IOC依赖注入**

按照spring上下文对实例化的Bean进行配置，完成IOC的注入；

**3.setBeanName设置BeanName实现**

如果这个Bean实现了BeanNameAware接口，会实现setBeanName方法，这个参数就是配置文件中Bean的Id值；

**4.BeanFactoryAware实现**

如果这个Bean已经实现了BeanFactoryAware接口，它会实现setBeanFactory方法，该方法传递的就是Spring工厂自身；可以通过这种方式获取其他的Bean信息；

**5.ApplicationContextAware实现**

如果这个Bean实现了ApplicationContextAware接口，它会实现setApplicationContext方法，该方法传递的是spring应用上下文接口ApplicationContext,比BeanFactory有更多的实现方法；

**6.postProcessBeforeInitialization，BeanPostProcess的方法，在初始化Bean之前执行**

如果这个Bean实现了BeanPostProcess接口，将会调用postProcessBeforeInitialzation(Object bean, String beanName)；

**7.init-method初始化方法**

如果这个Bean在spring配置文件中配置了init-method属性，则会自动调用其初始化方法。

**8.postProcessAfterInitialization，BeanPostProcess的方法，在初始化Bean之后执行**

完成以上流程之后就可以使用这个Bean了，这个Bean是Singleton的，当然也可以根据scope配置其他类型，prototype，session，request，globalsession；

**9.Destroy过期自动清理阶段**

当Bean不在需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy方法；

**10.destroy-method配置清理**

如果这个Bean配置了destroy-method属性，则会自动调用其配置的销毁方法；