如果方法被@Transactional标识，则此方法就会被spring的事务Aspect管理
在执行方法时会通过TransactionAspectSupport类的invokeWithinTransaction方法代理
第一次进入此方法会调用invokeWithinTransaction初始化事务连接，并将connection与dataSource key的绑定关系
存储到ThreadLocal<Map<Object, Object>> resources 中
堆栈如下
``` java
bindResource:178, TransactionSynchronizationManager (org.springframework.transaction.support)
doBegin:425, JpaTransactionManager (org.springframework.orm.jpa)
getTransaction:378, AbstractPlatformTransactionManager (org.springframework.transaction.support)
createTransactionIfNecessary:474, TransactionAspectSupport (org.springframework.transaction.interceptor)
invokeWithinTransaction:289, TransactionAspectSupport (org.springframework.transaction.interceptor)
invoke:98, TransactionInterceptor (org.springframework.transaction.interceptor)
proceed:185, ReflectiveMethodInvocation (org.springframework.aop.framework)
intercept:688, CglibAopProxy$DynamicAdvisedInterceptor (org.springframework.aop.framework)
insertSpring:-1, SymbolService$$EnhancerBySpringCGLIB$$f7383081 (top.vncnliu.event.server.mash.base.service.impl)
lambda$main$0:32, SymbolServiceTest (top.vncnliu.event.server.mash.sample.store.service.impl)
run:-1, 1464460851 (top.vncnliu.event.server.mash.sample.store.service.impl.SymbolServiceTest$$Lambda$767)
run:748, Thread (java.lang)
```
其他方法均通过
Connection con = DataSourceUtils.getConnection(obtainDataSource());获取连接，此方法最终从当前线程
的resources中获取连接，同一个连接，同一个事务
``` java
doGetResource:153, TransactionSynchronizationManager (org.springframework.transaction.support)
getResource:140, TransactionSynchronizationManager (org.springframework.transaction.support)
doGetConnection:103, DataSourceUtils (org.springframework.jdbc.datasource)
getConnection:78, DataSourceUtils (org.springframework.jdbc.datasource)
```

关于aop
spring aop通过动态代理实现，动态代理常用有两种
* 原生Proxy 必须有接口
* cglib 无需有接口，据说性能更高
原生示例代码
``` java
SomeInterface inter = (SomeInterface) Proxy.newProxyInstance(SomeInterfaceImpl.class.getClassLoader(),new Class[]{ SomeInterface.class },handler);
inter.doSomething();

class MyInvocationHandler implements InvocationHandler {

    //被代理对象
    private SomeInterfaceImpl target;
    private OtherObj otherObj;

    public MyInvocationHandler(SomeInterfaceImpl target, OtherObj otherObj) {
        this.target = target;
        this.otherObj = otherObj;
    }

    //这个proxy是代理对象，不要用作invoke的参数，会导致循环调用
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        otherObj.doSomething();
        return method.invoke(aopMain,args);
    }
}
```

这里有一个问题，同一个类的多个事务方法嵌套调用为什么不会导致多次初始化连接？
因为虽然嵌套调用多个方法但是只有入口方法才会触发代理类的invoke调用。
动态代理的思想就是动态生成一个类（静态代理这个类就要我们自己写了），并重写类的方法。
通过System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true");
将生成的代理类保存反编译
``` java
// 原始
public void doSomething();
descriptor: ()V
flags: ACC_PUBLIC
Code:
  stack=2, locals=1, args_size=1
     0: getstatic     #14                 // Field java/lang/System.out:Ljava/io/PrintStream;
     3: ldc           #15                 // String do in main
     5: invokevirtual #16                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
     8: aload_0
     9: invokevirtual #17                 // Method doSomthing2:()V
    12: return
  LineNumberTable:
    line 27: 0
    line 28: 8
    line 29: 12

// 代理类
  public final void doSomething() throws ;
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_FINAL
    Code:
      stack=10, locals=2, args_size=1
         0: aload_0
         1: getfield      #16                 // Field java/lang/reflect/Proxy.h:Ljava/lang/reflect/InvocationHandler;
         4: aload_0
         5: getstatic     #57                 // Field m4:Ljava/lang/reflect/Method;
         8: aconst_null
         9: invokeinterface #28,  4           // InterfaceMethod java/lang/reflect/InvocationHandler.invoke:(Ljava/lang/Object;Ljava/lang/reflect/Method;[Ljava/lang/Object;)Ljava/lang
/Object;
        14: pop
        15: return
        16: athrow
        17: astore_1
        18: new           #42                 // class java/lang/reflect/UndeclaredThrowableException
        21: dup
        22: aload_1
        23: invokespecial #45                 // Method java/lang/reflect/UndeclaredThrowableException."<init>":(Ljava/lang/Throwable;)V
        26: athrow
      Exception table:
         from    to  target type
             0    16    16   Class java/lang/Error
             0    16    16   Class java/lang/RuntimeException
             0    16    17   Class java/lang/Throwable
    Exceptions:
      throws
```
可以看出invoke生成的类将原始方法的调用改为调用invoke方法，了解之后逻辑还是很简单清晰的。