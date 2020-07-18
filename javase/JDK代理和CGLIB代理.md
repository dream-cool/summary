# JDK静态代理

静态代理就是在程序运行之前，代理类字节码 `.class`就已编译好，通常一个静态代理类也只代理一个目标类，代理类和目标类都实现相同的接口。接下来就先通过 demo 进行分析什么是静态代理，当前创建一个 Animal 接口，里面包含 `call`函数。

```java
package other.proxy.jdk;

public interface Animal {	
    void call();
  
}
```

目标类 Cat

```java
package other.proxy.jdk;

/**
 * @author clt
 * @create 2020/5/24 10:00
 */
public class Cat implements Animal {
    @Override
    public void call() {
        System.out.println("喵喵喵  ");
    }
}

```

目标类 Dog

```java
package other.proxy.jdk;

/**
 * @author clt
 * @create 2020/5/24 10:06
 */
public class Dog implements  Animal {
    @Override
    public void call() {
        System.out.println("汪汪汪----");
    }
}
```

通过代理类实现目标类的前后事件调用，达到代理作用。

```java
package other.proxy.jdk;

/**
 * @author clt
 * @create 2020/5/24 10:02
 */
public class StaticProxyAnimal implements Animal{

    private Animal impl;

    public StaticProxyAnimal(Animal impl){
        this.impl = impl;
    }

    @Override
    public void call() {
        beforeEvents();
        impl.call();
        afterEvents();
    }

    private void beforeEvents(){
        System.out.println("饥饿了-------（前置增强事件）");
    }

    private void afterEvents(){
        System.out.println("找吃的-------（后置增强事件）");
    }


}

```

测试

```java
package other.proxy.jdk;

import org.junit.Test;

/**
 * @author clt
 * @create 2020/5/24 10:04
 */
public class Main {

    @Test
    public void testStaticProxy(){
        Animal staticProxy = new StaticProxyAnimal(new Cat());
        staticProxy.call();
    }

    @Test
    public  void test(){
        Animal staticProxy = new StaticProxyAnimal(new Dog());
        staticProxy.call();
    }
}

```

结果



# JDK动态代理

动态代理类与静态代理类最主要不同的是，代理类的字节码不是在程序运行前生成的，而是在程序运行时再虚拟机中程序自动创建的。



获取代理对象

```java
package other.proxy.jdk;

import java.lang.reflect.Proxy;

/**
 * @author clt
 * @create 2020/5/24 10:38
 */
public class DynamicProxyAnimal {

    public  static Object getProxy(Object target) throws Exception{
        /**
         * loader 加载代理对象的类加载器
         * interfaces 代理对象实现的接口，与目标对象实现同样的接口
         * h 处理代理对象逻辑的处理器，即上面的 InvocationHandler 实现类。
         */
        Object proxy = Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new TargetInvoker(target)
        );
        return proxy;
    }

}

```



代理类

```java
package other.proxy.jdk;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author clt
 * @create 2020/5/24 10:34
 */
public class TargetInvoker implements InvocationHandler {

    private Object target;

    public TargetInvoker(Object target){
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        beforeEvents();
        Object result = method.invoke(target, args);
        afterEvents();
        return result;
    }

    private void beforeEvents(){
        System.out.println("JDK代理前-------（前置增强事件）");
    }

    private void afterEvents(){
        System.out.println("JDK代理后-------（后置增强事件）");
    }


}

```

测试

```java
	@Test
    public  void  testJDKDynamicProxy1() throws Exception {
        Animal proxy = (Animal) DynamicProxyAnimal.getProxy(new Cat());
        proxy.call();
    }

    @Test
    public  void  testJDKDynamicProxy2() throws Exception {
        Animal proxy = (Animal) DynamicProxyAnimal.getProxy(new Dog());
        proxy.call();
    }
```



生成的代理对象class文件

```java

package com.sun.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;
import other.proxy.Animal;

public final class $Proxy4 extends Proxy implements Animal {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy4(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void call() throws  {
        try {
            //在构造函数中 拿到传过来的 InvocationHandler 
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("other.proxy.Animal").getMethod("call");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```







####参考 https://mp.weixin.qq.com/s?__biz=MzAwOTE3NDY5OA==&mid=2647911673&idx=6&sn=9e4c61bc6f3b82ab165e8985c9653916&chksm=8344e73cb4336e2ab62eeeff9d5784eec27035eb71db8093d76045b77b220f8ad5dfacf40b7c&mpshare=1&scene=24&srcid=&sharer_sharetime=1590282754796&sharer_shareid=0e96b1c3488474918e14a4881f44083a#rd