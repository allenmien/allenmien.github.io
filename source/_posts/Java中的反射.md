---
title:      Java中的反射
---
# 反射机制
## 理解Class类

- Class是一个类。
- Class是用来描述类的类。
- Class是一个类，封装了当前对象所对应的类的信息。
- 一个类中有属性，方法，构造器等，比如说有一个Person类，现在需要一个类，用来描述类，这就是Class，它应该有Person的类名，属性，方法，构造器等。
- Class类：是一个对象照镜子的结果，对象可以看到自己有哪些属性，方法，构造器，实现了哪些接口等等。
- 对于每个类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个类的有关信息。 
- Class 对象：只能由系统建立对象，一个类（而不是一个对象）在 JVM 中只会有一个Class实例。

### 获取Class对象的三种方式

1. 通过类名获取      类名.class
2. 通过对象获取      对象名.getClass()
3. 通过全类名获取    Class.forName(全类名)



**Person类：**

```java
public class Person {
    private String name;
    private int age;

    //新增一个私有方法
    private void privateMthod() {
    }

    public Person() {
        System.out.println("无参构造器");
    }

    public Person(String name, int age) {
        System.out.println("有参构造器");
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    /**
     * @param age 类型用Integer，不用int
     */
    public void setName(String name, int age) {
        System.out.println("name: " + name);
        System.out.println("age:" + age);

    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

**ReflectionTest：**

```java
public class ReflectionTest {
    public static void main(String[] args) throws ClassNotFoundException {
        Class clazz = null;
        //1.通过类名
        clazz = Person.class;

        //2.通过对象名
        //这种方式是用在传进来一个对象，却不知道对象类型的时候使用
        Person person = new Person();
        clazz = person.getClass();
        //上面这个例子的意义不大，因为已经知道person类型是Person类，再这样写就没有必要了
        //如果传进来是一个Object类，这种做法就是应该的
        Object obj = new Person();
        clazz = obj.getClass();

        //3.通过全类名(会抛出异常)
        //一般框架开发中这种用的比较多，因为配置文件中一般配的都是全类名，通过这种方式可以得到Class实例
        String className="Person";
        clazz = Class.forName(className);

        //字符串的例子
        clazz = String.class;
        clazz = "javaTest".getClass();
        clazz = Class.forName("java.lang.String");
        System.out.println();
    }
}
```

### Class类的常用方法

**ReflectionTest:**

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ReflectionTest {
    public static void main(String[] args) throws ClassNotFoundException {
        Class clazz = null;

        Object obj = new Person();
        clazz = obj.getClass();

        //返回此Class对象所表示的实体（类、接口、数组类、基本类型或void）名称
        System.out.println("------------>getName<------------");
        System.out.println("获取此Class对象所表示的实体（类、接口、数组类、基本类型或void）名称：");
        System.out.println(clazz.getName());

        //
        System.out.println("------------>getConstructors<------------");
        System.out.println("获取Person的构造方法：");
        Constructor<Person>[] constructors = (Constructor<Person>[]) Class.forName("Person").getConstructors();
        for (Constructor<Person> constructor : constructors) {
            System.out.println(constructor);
        }

        //获取Person的方法名
        System.out.println("------------>getMethods<------------");
        System.out.println("获取Person的方法名：");
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            System.out.println(method.getName());
        }

        //获取Person的属性
        System.out.println("------------>getDeclaredFields<------------");
        System.out.println("获取Person的属性：");
        Field[] field = clazz.getDeclaredFields();
        for (Field f : field) {
            System.out.println(f.getName());
        }

        System.out.println();
    }
}

```

**result:**

```
------------>getName<------------
获取此Class对象所表示的实体（类、接口、数组类、基本类型或void）名称：
Person
------------>getConstructors<------------
获取Person的构造方法：
public Person(java.lang.String,int)
public Person()
------------>getMethods<------------
获取Person的方法名：
toString
getName
setName
setName
getAge
setAge
wait
wait
wait
equals
hashCode
getClass
notify
notifyAll
------------>getDeclaredFields<------------
获取Person的属性：
name
age
```

**newInstance（）方法**

- 通过反射创建实例的时候，实际调用的是类的**无参数的构造器**。
- 所以在我们在定义一个类的时候，定义一个有参数的构造器，作用是对属性进行初始化，还要写一个无参数的构造器，作用就是反射时候用。
- 一般地、一个类若声明一个带参的构造器，同时要声明一个无参数的构造器

**ReflectionTest:**

```java
public class ReflectionTest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Class clazz = null;

        clazz = Class.forName("Person");
        Object object = clazz.newInstance();

        System.out.println(object);
    }
}
```

**result:**

```
无参构造器
Person{name='null', age=0}
```

## ClassLoader

类装载器是用来把类(class)装载进 JVM 的。

JVM在运行时会产生3个类加载器组成的初始化加载器层次结构 ，如下图所示：

![](https://images0.cnblogs.com/blog/534926/201401/181711178455.jpg)

```java
public class ReflectionTest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        //1. 获取一个系统的类加载器(可以获取，当前这个类PeflectTest就是它加载的)
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        System.out.println(classLoader);


        //2. 获取系统类加载器的父类加载器（扩展类加载器，可以获取）.
        classLoader = classLoader.getParent();
        System.out.println(classLoader);


        //3. 获取扩展类加载器的父类加载器（引导类加载器，不可获取）.
        classLoader = classLoader.getParent();
        System.out.println(classLoader);


        //4. 测试当前类由哪个类加载器进行加载（系统类加载器）:
        classLoader = Class.forName("ReflectionTest")
                .getClassLoader();
        System.out.println(classLoader);


        //5. 测试 JDK 提供的 Object 类由哪个类加载器负责加载（引导类）
        classLoader = Class.forName("java.lang.Object")
                .getClassLoader();
        System.out.println(classLoader);
    }
}
```

result:

```
jdk.internal.loader.ClassLoaders$AppClassLoader@4f8e5cde
jdk.internal.loader.ClassLoaders$PlatformClassLoader@16f65612
null
jdk.internal.loader.ClassLoaders$AppClassLoader@4f8e5cde
null
```

## 反射

反射  是Java被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的內部信息，并能直接操作任意对象的内部属性及方法。

Java反射机制主要提供了以下功能：

- 在运行时构造任意一个类的对象
- 在运行时获取任意一个类所具有的成员变量和方法
- 在运行时调用任意一个对象的方法（属性）
- 生成动态代理

### 获取方法

#### getMethods

- 不能获取private方法
- 会获取从父类继承来的所有方法

#### getDeclaredMethods

- 可以获取private方法
- 不能获取从父类继承来的方法，只能获取本类

#### getDeclaredMethod

- 获取指定名称的方法

#### invoke

- 执行某个对象的方法，改变对象的属性值

ReflectionTest：

```java
import java.lang.reflect.Method;

public class ReflectionTest {
    public static void main(String args[]) throws Exception{
        Class clazz = Class.forName("Person");

        //1.获取方法
        //  1.1 获取取clazz对应类中的所有方法--方法数组（一）
        //     不能获取private方法,且获取从父类继承来的所有方法
        System.out.println("--------------getMethods--------------");
        Method[] methods = clazz.getMethods();
        for(Method method:methods){
            System.out.println(" "+method.getName());
        }
        System.out.println();

        //  1.2.获取所有方法，包括私有方法 --方法数组（二）
        //  所有声明的方法，都可以获取到，且只获取当前类的方法
        System.out.println("--------------getDeclaredMethods--------------");
        methods = clazz.getDeclaredMethods();
        for(Method method:methods){
            System.out.println(" "+method.getName());
        }
        System.out.println();

        //  1.3.获取指定的方法
        //  需要参数名称和参数列表，无参则不需要写
        //  对于方法public void setName(String name) {  }
        Method method = clazz.getDeclaredMethod("setName", String.class);
        System.out.println(method);
        //  而对于方法public void setAge(int age) {  }
        method = clazz.getDeclaredMethod("setAge", Integer.class);
        System.out.println(method);
        //  这样写是获取不到的，如果方法的参数类型是int型
        //  如果方法用于反射，那么要么int类型写成Integer： public void setAge(Integer age) {  }

        //2.执行方法
        //  invoke第一个参数表示执行哪个对象的方法，剩下的参数是执行方法时需要传入的参数
        System.out.println("--------------invoke--------------");
        Object obje = clazz.newInstance();
        System.out.println("setAge : 2" );
        method.invoke(obje,2);
        Method getAgeMethod = clazz.getDeclaredMethod("getAge");
        int age = (int) getAgeMethod.invoke(obje);
        System.out.println("getAge : " + String.valueOf(age));
    }
}
```

控制台：

```java
--------------getMethods--------------
 toString
 getName
 setName
 setName
 setAge
 getAge
 wait
 wait
 wait
 equals
 hashCode
 getClass
 notify
 notifyAll

--------------getDeclaredMethods--------------
 toString
 getName
 setName
 setName
 privateMthod
 setAge
 getAge

public void Person.setName(java.lang.String)
public void Person.setAge(java.lang.Integer)
--------------invoke--------------
无参构造器
setAge : 2
getAge : 2
```

#### 方法反射调用工具

这种反射实现的主要功能是可配置和低耦合。只需要类名和方法名，而不需要一个类对象就可以执行一个方法。如果我们把全类名和方法名放在一个配置文件中，就可以根据调用配置文件来执行方法

##### 对象和方法名做参数

main:

```java
import java.lang.reflect.Method;

public class Test {
    public static void main(String[] args) throws Exception {
        Object obj = new Person();
        invoke(obj, "test", "wang", 1);
    }

    public static Object invoke(Object obj, String methodName, Object... args) throws Exception{
        //1. 获取 Method 对象
        //   因为getMethod的参数为Class列表类型，所以要把参数args转化为对应的Class类型。

        Class [] parameterTypes = new Class[args.length];
        for(int i = 0; i < args.length; i++){
            parameterTypes[i] = args[i].getClass();
            System.out.println(parameterTypes[i]);
        }

        Method method = obj.getClass().getDeclaredMethod(methodName, parameterTypes);
        //如果使用getDeclaredMethod，就不能获取父类方法，如果使用getMethod，就不能获取私有方法

        //2. 执行 Method 方法
        //3. 返回方法的返回值
        return method.invoke(obj, args);
    }
}
```

Person:

```java
public Class Person{
    public void test(String name,Integer age){
        System.out.println("调用成功");
    }
}
```

控制台：

```
调用成功
```

##### 类名和方法名做参数

main:

```java
import java.lang.reflect.Method;

public class Test {
    public static void main(String[] args) throws Exception {
        invoke("Person", "test", "wang", 1);
    }

    public static Object invoke(String className, String methodName, Object... args) throws Exception{
        Object obj = null;
        obj = Class.forName(className).newInstance();

        //1. 获取 Method 对象
        //   因为getMethod的参数为Class列表类型，所以要把参数args转化为对应的Class类型。

        Class [] parameterTypes = new Class[args.length];
        for(int i = 0; i < args.length; i++){
            parameterTypes[i] = args[i].getClass();
            System.out.println(parameterTypes[i]);
        }

        Method method = obj.getClass().getDeclaredMethod(methodName, parameterTypes);
        //如果使用getDeclaredMethod，就不能获取父类方法，如果使用getMethod，就不能获取私有方法

        //3. 返回方法的返回值
        return method.invoke(obj, args);
    }
}
```

##### 

