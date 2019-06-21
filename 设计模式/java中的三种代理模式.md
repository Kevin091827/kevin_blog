上文中我总结过代理模式，代理模式是一种可以在不改变目标对象源码的情况下，实现对目标对象的增强[代理模式](https://blog.csdn.net/weixin_41922289/article/details/92842725)

具体分为：

- 静态代理

- 动态代理
    - jdk动态代理
    - cglib动态代理

>今天来看看java中的代理模式的经典的三种使用

# 一，静态代理
静态代理是代理模式的实现方式之一，是相对于动态代理而言的。所谓静态代理是指，在程序运行前，由程序员创建或特定工具自动生成源代码并对其编译生成.class文件。静态代理的实现只需要三步：首先，定义业务接口；其次，实现业务接口；然后，定义代理类并实现业务接口；最后便可通过客户端进行调用


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

## 1.相关类
在动态代理中，代理类是在运行时期生成的。因此，相比静态代理，动态代理可以很方便地对委托类的相关方法进行统一增强处理，如添加方法调用次数、添加日志功能等等。动态代理主要分为JDK动态代理和cglib动态代理两大类


要想使用JDK动态代理，首先需要了解其相关的类或接口：

java.lang.reflect.Proxy：该类用于动态生成代理类，只需传入目标接口、目标接口的类加载器以及InvocationHandler便可为目标接口生成代理类及代理对象。

```java
// 方法 1: 该方法用于获取指定代理对象所关联的InvocationHandler
static InvocationHandler getInvocationHandler(Object proxy) 

// 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象
static Class getProxyClass(ClassLoader loader, Class[] interfaces) 

// 方法 3：该方法用于判断指定类是否是一个动态代理类
static boolean isProxyClass(Class cl) 

// 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例
static Object newProxyInstance(ClassLoader loader, Class[] interfaces, 
    InvocationHandler h)
```

java.lang.reflect.InvocationHandler：该接口包含一个invoke方法，通过该方法实现对委托类的代理的访问，是代理类完整逻辑的集中体现，包括要切入的增强逻辑和进行反射执行的真实业务逻辑。

```java
// 该方法代理类完整逻辑的集中体现。第一个参数既是代理类实例，第二个参数是被调用的方法对象，第三个方法是调用参数。通常通过反射完成对具体角色业务逻辑的调用，并对其进行增强。
Object invoke(Object proxy, Method method, Object[] args)
```
## 2.实现
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

## 3.分析
 Proxy.newProxyInstance方法源码：
 ```java
     @CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);
        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }
        Class<?> cl = getProxyClass0(loader, intfs);
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
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