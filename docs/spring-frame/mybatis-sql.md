# Mybatis执行Sql过程？

1.SqlSessionFactoryBuilder去读取mybatis的配置文件，然后build一个DefaultSqlSessionFactory；

2.通过SqlSessionFactory获取SqlSession对象；

3.通过SqlSession.getMapper()根据传入的Mapper对象类型动态代理并返回一个动态代理后的Mapper对象；

4.由SqlSession.select()/update()，MapperProxy对象的invoke()方法执行后再执行execure方法，再根据情况选择执行select/update；

5.Executor执行Query/queryFromDatabase，在前面经过参数名封装和缓存查询之后（缓存为空），会调用queryFromDatabase方法去数据库当中查；

6.SimpleExecurot执行doQuery()方法，初始化prepareStatement并且给#{}参数赋值

7.StatementHandler执行query()方法，执行sql语句

8.ResuletHandler.handleResultSets()方法封装结果集