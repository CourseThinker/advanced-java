1、spring的事务注解@Transactional只能放在public修饰的方法上才起作用，如果放在其他非public（private，protected）方法上，虽然不报错，但是事务不起作用

2、如果采用spring+spring mvc，则context:component-scan重复扫描问题可能会引起事务失败。 

        如果spring和mvc的配置文件中都扫描了service层，那么事务就会失效。
    
       原因：因为按照spring配置文件的加载顺序来讲，先加载springmvc配置文件，再加载spring配置文件，我们的事物一般都在srping配置文件中进行配置，如果此时在加载srpingMVC配置文件的时候，把servlce也给注册了，但是此时事物还没加载，也就导致后面的事物无法成功注入到service中。所以把对service的扫描放在spring配置文件中或是其他配置文件中。


3、如使用mysql且引擎是MyISAM，则事务会不起作用，原因是MyISAM不支持事务，可以改成InnoDB引擎

4、 @Transactional 注解开启配置，必须放到listener里加载，如果放到DispatcherServlet的配置里，事务也是不起作用的。

![img](https://img-blog.csdnimg.cn/20190308174650466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Roa2xzbA==,size_16,color_FFFFFF,t_70)

5、Spring团队建议在具体的类（或类的方法）上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上。在接口上使用 @Transactional 注解，只能当你设置了基于接口的代理时它才生效。因为注解是 不能继承 的，这就意味着如果正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装。

6、在业务代码中如果抛出RuntimeException异常，事务回滚；但是抛出Exception，事务不回滚；

7、如果在加有事务的方法内，使用了try...catch..语句块对异常进行了捕获，而catch语句块没有throw  new RuntimeExecption异常，事务也不会回滚;

原文链接：https://blog.csdn.net/dhklsl/article/details/88354216