上文中我总结过代理模式，代理模式是一种可以在不改变目标对象源码的情况下，实现对目标对象的增强[代理模式](https://blog.csdn.net/weixin_41922289/article/details/92842725)

>今天来看看java中的代理模式的经典的三种使用

# 一，静态代理

抽象者：
```java
public interface Rent {

    void rentHouse();
}
```
目标对象：
```java
public class Host implements Rent {

    @Override
    public void rentHouse() {
        System.out.println("-------> 房东出租房子");
    }
}
```
代理对象:
```java
public class Proxy implements Rent {

    private Rent rent;

    public Proxy(Rent rent){
        this.rent = rent;
    }

    @Override
    public void rentHouse() {
        rent.rentHouse();
    }
}
```
测试：
```java
    public static void main(String[] args) {

        Host host = new Host();
        Proxy proxy = new Proxy(host);
        proxy.rentHouse();
    }
```

# 二，动态代理

抽象者：
```java
public interface Target {

    void targetMethod();
}

```
目标对象：
```java
public class TargetClass implements Target {

    @Override
    public void targetMethod() {
        System.out.println("目标类实现目标方法");
    }
}
```


代理：
```java
public class ProxyClass {

    public static Target createProxy(){
        final Target targetClass = new TargetClass();

        final Aspect aspect = new Aspect();

        Target proxy = (Target) Proxy.newProxyInstance(ProxyClass.class.getClassLoader(), targetClass.getClass().getInterfaces(),new InvocationHandler() {

            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
               aspect.before();

               Object result = method.invoke(targetClass,args);

               aspect.after();

               return result;
            }
        });
        return proxy;
    }
}
```

测试：
```java
    public static void main(String[] args) {
        Target target = ProxyClass.createProxy();
        target.targetMethod();
    }
```

# 三，cglib代理
抽象者：
```java
public interface ITarget {

    void targetMethod();
}
```

目标对象
```java
public class Target implements ITarget{

    @Override
    public void targetMethod(){
        System.out.println("--------> 目标方法");
    }
}
```


代理对象：
```java
public class Proxy implements MethodInterceptor {

    private Enhancer enhancer = new Enhancer();

    public Object getProxy(Class clazz){
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("前置代理");
        //通过代理类调用父类中的方法
        Object result = methodProxy.invokeSuper(o, objects);
        System.out.println("后置代理");
        return result;
    }
}
```

测试：
```java
    public static void main(String[] args) {
        Proxy proxy = new Proxy();
        Target target = (Target) proxy.getProxy(ITarget.class);
        target.targetMethod();
    }
```