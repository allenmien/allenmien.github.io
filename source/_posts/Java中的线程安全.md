---
title:      Java中的线程安全
---
# 线程安全

什么是线程安全？

> 通俗点说，就是线程访问时不产生资源冲突。
>
> “一个类可以被多个线程安全调用就是线程安全的”
>
> ​																————《Java编程并发实践》

## 变量的线程安全

### 静态变量：线程非安全

用public修饰的static成员变量和成员方法本质是变量和全局方法，当声明它的类的对象时，不生成static变量的副本，而是类的所有实例共享同一个static变量。

静态变量也称为类变量，属于类对象所有，位于方法区，为所有对象共享，共享一份内存，一旦值被修改，则其他对象均对修改可见，故线程非安全。

```java
/**
 * Created by Mark on 2018/1/5.
 */
public class HelloInstance implements Runnable {
    private static int num;

    @Override
    public void run() {
        num = 3;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num);
        num = 5;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num * 2);

    }

    public static void main(String[] args) {
        for (int i = 1; i <= 50; i++) {
            new Thread(new HelloInstance(), "Thread=" + 1).start();
        }
    }
}
```

如果线程安全，应该只会有3和10。但是出现了5的情况。说明线程非安全。

result:

```
当前线程是：Thread=1,num的值是：3
当前线程是：Thread=1,num的值是：10
当前线程是：Thread=1,num的值是：5
```

### 实例变量：单例时线程非安全，非单例时线程安全

实例变量是实例对象私有的，系统只存在一个实例对象，则在多线程环境下，如果值改变后，则其它对象均可见，故线程非安全；

如果每个线程都在不同的实例对象中执行，则对象与对象间的修改互不影响，故线程安全。

#### 单例

```java
/**
 * Created by Mark on 2018/1/5.
 */
public class HelloInstance implements Runnable {
    private int num;

    @Override
    public void run() {
        num = 3;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num);
        num = 5;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num * 2);

    }

    public static void main(String[] args) {
        HelloInstance helloInstance = new HelloInstance();
        for (int i = 1; i <= 50; i++) {
            new Thread(helloInstance, "Thread=" + 1).start();
        }
    }
}
```

result：非安全

```
当前线程是：Thread=1,num的值是：10
当前线程是：Thread=1,num的值是：5
当前线程是：Thread=1,num的值是：3
```

#### 多例

```java
/**
 * Created by Mark on 2018/1/5.
 */
public class HelloInstance implements Runnable {
    private int num;

    @Override
    public void run() {
        num = 3;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num);
        num = 5;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num * 2);

    }

    public static void main(String[] args) {
        for (int i = 1; i <= 50; i++) {
            new Thread(new HelloInstance(), "Thread=" + 1).start();
        }
    }
}
```

result：线程安全

```
当前线程是：Thread=1,num的值是：3
当前线程是：Thread=1,num的值是：3
当前线程是：Thread=1,num的值是：10
当前线程是：Thread=1,num的值是：10
当前线程是：Thread=1,num的值是：3
```

### 局部变量：线程安全

每个线程执行时都会把局部变量放在各自的帧栈的内存空间中，线程间不共享，故不存在线程安全问题。

```java
/**
 * Created by Mark on 2018/1/5.
 */
public class HelloInstance implements Runnable {

    @Override
    public void run() {
        int num = 3;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num);
        num = 5;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + num * 2);

    }

    public static void main(String[] args) {
        HelloInstance helloInstance = new HelloInstance();
        for (int i = 1; i <= 50; i++) {
            new Thread(helloInstance, "Thread=" + 1).start();
        }
    }
}
```

result:

```
当前线程是：Thread=1,num的值是：3
当前线程是：Thread=1,num的值是：10
当前线程是：Thread=1,num的值是：3
当前线程是：Thread=1,num的值是：10
```

### 静态方法的线程安全性

静态方法中如果没有使用静态变量，则没有线程安全的问题；

- Java是线程安全的，即对任何方法（包括静态方法）都可以不考虑线程冲突，但有一个前提，就是不能存在全局变量。如果存在全局变量，则需要使用同步机制。
- Java在执行静态方法时，如果静态方法所在的类里面没有静态的变量，那么线程访问就是安全的。
- Java在执行静态方法时，如果使用静态变量，同时类的函数设计时使用到了静态数据，最好在调用函数时使用synchronized关键字，否则会导致数据的不一致行。
- 加静态全局的变量，在多线程访问下定会出现数据的不一致行，最好使用synchronized关键字，确保数据的一致性，典型的代表就是单例模式。

#### 静态方法不使用静态变量

线程安全

StaticThread：

```java
/**
 * Created by Mark on 2018/1/6.
 */
public class StaticThread implements Runnable {
    @Override
    public void run() {
        StaticAction.print();
    }
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(new StaticThread()).start();
        }
    }
}
```

StaticAction：

```java
/**
 * Created by Mark on 2018/1/6.
 */
public class StaticAction {
    public static int i = 0;
    public static void print() {
        int sum = 0;
        for (int i = 0; i < 10; i++) {
            System.out.print("step " + i + " is running.");
            sum += i;
        }
        if (sum != 45) {
            System.out.println("Thread error!");
            System.exit(0);
        }
        System.out.println("sum is " + sum);
    }
}
```

结果：线程安全

```
没有出现：Thread error!
```

#### 静态方法使用静态变量

有可能会产生线程不安全

线程不安全代码：

StaticThread：

```java
/**
 * Created by Mark on 2018/1/6.
 */
public class StaticThread implements Runnable {
    @Override
    public void run() {
        StaticAction.incValue();
    }
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(new StaticThread()).start();
        }
        try {
            Thread.sleep(1000);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(StaticAction.i);
    }
}
```

StaticAction:

```java
/**
 * Created by Mark on 2018/1/6.
 */
public class StaticAction {
    public static int i = 0;
    public static void incValue() {
        int temp = StaticAction.i;
        try {
            Thread.sleep(1);
        } catch (Exception e) {
            e.printStackTrace();
        }
        temp++;
        StaticAction.i = temp;
    }
}
```

第一次运行结果：

```
16
```

第二次运行结果：

```
11
```

线程安全代码（synchronized）：

```java
/**
 * Created by Mark on 2018/1/6.
 */
public class StaticAction {
    public static int i = 0;
    public static void incValue() {
        int temp = StaticAction.i;
        try {
            Thread.sleep(1);
        } catch (Exception e) {
            e.printStackTrace();
        }
        temp++;
        StaticAction.i = temp;
    }
}
```

结果：

```
100
```



## 经验

### 经验一

Hello_Main:

```java
/**
 * Created by Mark on 2018/1/4.
 */
public class Hello_Main {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
        Employee employee = new Employee();
        employee.reload();
        for (int i = 1; i <= 50; i++) {
            new Thread(new HelloInstance(), "Thread=" + 1).start();
        }
    }
}
```
HelloInstance:
```java
/**
 * Created by Mark on 2018/1/5.
 */
public class HelloInstance implements Runnable {

    @Override
    public void run() {
        ExtraProcessor GKprocessor = Employee.configMap.get("GKprocessor");
        int num = 3;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + String.valueOf(GKprocessor.getProcessor(num)));
        num = 5;
        System.out.println("当前线程是：" + Thread.currentThread().getName() + ",num的值是：" + String.valueOf(GKprocessor.getProcessor(num)));
    }
}
```

Employee:

```java
import java.util.HashMap;
import java.util.Map;

public class Employee{
     public static Map<String, ExtraProcessor> configMap = new HashMap<>();

     public void reload() throws ClassNotFoundException, IllegalAccessException, InstantiationException {
         configMap.put("GKprocessor", (ExtraProcessor) Class.forName("GKprocessor").newInstance());
     }
}
```

ExtraProcessor:

```java
/**
 * Created by Mark on 2018/1/6.
 */
public interface ExtraProcessor {
    int getProcessor(int a);
}

```

GKprocessor:

```java
/**
 * Created by Mark on 2018/1/6.
 */
public class GKprocessor implements ExtraProcessor{
    @Override
    public int getProcessor(int a) {
        return a;
    }
}
```

