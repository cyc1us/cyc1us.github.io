---
layout: post
title: "从反射出发"
date: 2024-03-20 
description: "Java反射及其延申出来的知识"
tag: Java
--- 
​最近在学习Spring源码的过程中，像反射这些概念会被大量地提起。但你要说这些知识吧知道是肯定知道的，要让说出来个一二也能说一下，但就是感觉这些知识离自己挺远的，没有什么归属感。总结了一下还是用得少，听得也少。所以说为了巩固这种类型的知识我一般就会写一篇文章来记录它，并且从反射还能引出很多其它的知识，就当是我这种不爱做笔记的同学的一个笔记。

## 反射

​反射是指在程序运行时动态获取类的信息，以及动态调用对象的方法。在许多编程语言中，反射提供了一种强大的机制，使得开发者能够在运行时查看类的属性、方法和构造函数，甚至可以动态创建对象和调用方法。在Java中，反射机制主要通过java.lang.reflect包来实现，主要功能为访问类的字段、方法、构造函数，以及创建类的实例。反射也有以下几点需要注意：

- **性能影响**：由于反射是动态的，因此会比直接调用慢，频繁使用可能会影响性能。
- **安全性**：反射可以访问私有字段和方法，可能会破坏封装性，所以使用时要小心。
- **复杂性**：代码的可读性和可维护性可能会降低。

### 获取class对象

在java中，每一个类都有一个与之关联的Class对象，而其实反射的功能包括获取类的构造方法、获取类的属性的基础首先都是要获取到这个class对象。获取class对象有3种方法：

- Class.forName("全类名")
- 类名.class
- 对象.getClass()

假如我有如下MyClass对象：
~~~ java
public class MyClass {
    private String name;
    private int age;

    public MyClass() {
    }

    public MyClass(String name, int age) {
        this.name = name;
        this.age = age;
    }

    /*省略修改访问方法*/
}
~~~

那么我就可以用以下代码来获取Class对象：
~~~ java
public class ReflectionDemo {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        // 方法1：使用 Class.forName() 方法
        Class<?> class1 = Class.forName("reflect.MyClass");
        System.out.println("通过 Class.forName() 获取的类名: " + class1.getName());

        // 方法2：使用 .class 属性
        Class<MyClass> class2 = MyClass.class;
        System.out.println("通过 .class 获取的类名: " + class2.getName());

        // 方法3：使用 getClass() 方法
        MyClass myClassInstance = new MyClass();
        Class<?> class3 = myClassInstance.getClass();
        System.out.println("通过 getClass() 获取的类名: " + class3.getName());
    }
}
~~~

其运行结果如下：

~~~ 
通过 Class.forName() 获取的类名: reflect.MyClass
通过 .class 获取的类名: reflect.MyClass
通过 getClass() 获取的类名: reflect.MyClass
~~~

那么现在我们已经获取到了类的class对象，就可以利用这个对象来获取类的其它属性了

### 获取构造方法

获取构造方法首先要通过上述方式获取到Class对象，之后你有如下四种选择来获取构造方法：

- Constructor<?>[] getConstructors()：返回一个Constructor数组，包括所有公开的构造方法
- Constructor<?>[] getDeclaredConstructors()：返回一个Constructor数组，包括所有构造方法（无论是公共的、私有的、还是受保护的）
- Constructor<\T> getConstructor(Class<?>... parameterTypes)：返回指定参数类型的公共构造方法对象
- Constructor<\T> getDeclaredConstructor(Class<?>... parameterTypes):返回指定参数类型的构造方法对象，可能是公共的、私有的或受保护的

<img src="/images/posts/Java/reflect/image-20240920155158696.png" alt="image-20240920155158696" style="zoom: 67%;" />

~~~ java
Constructor<?>[] constructors = clazz.getConstructors();
        
Constructor<?>[] declaredConstructors = clazz.getDeclaredConstructors();
        
Constructor<MyClass> declaredConstructor = clazz.getDeclaredConstructor(String.class,int.class);
~~~

假如说我现在有上面四个构造方法，并且将得到的三个构造对象输出会得到以下结果：
~~~ java
public reflect.MyClass(java.lang.String)
public reflect.MyClass()
-------------------------------------------
private reflect.MyClass(java.lang.String,int)
protected reflect.MyClass(int)
public reflect.MyClass(java.lang.String)
public reflect.MyClass()
-------------------------------------------
private reflect.MyClass(java.lang.String,int)
~~~

可以看出只有只有带上Declared的方法才能获取所有构造方法。并且在使用获取单个构造器方法的时候，入参要与构造方法的入参一致。

那么获取了构造方法时候还可以干什么呢？可以通过constructor.getModifiers()来获取构造方法的权限修饰符，这个方法的返回值是一个int类型的整数，不同权限的修饰符对应的值是不同的，这是jdk底层所定义的，不过需要注意，默认（包）访问权限没有直接的常量对应。其实现逻辑是依赖编译器而非运行时常量。

那么既然得到了一个类的构造器，我们也可以利用这个构造器来创建对象，方法为`constructor.newInstance(参数)`，这里的参数要和构造器的参数一致。有个问题需要注意，如果说你的构造器私有的或者受保护的，那么你直接用这个方法来创建对象是会报错的：

![image-20240920160946374](/images/posts/Java/reflect/image-20240920160946374.png)

如果说你还是想要用这个方法来构造对象，那可以用`constructor.setAccessibe(true)`来临时取消权限的校验，这样就可以成功创建对象了

### 获取成员变量

获取成员变量的方法跟获取构造器的方法除了名字不一样，逻辑基本上是一样的，有四个：

- Field[] getFields()：返回一个包含Class对象所表示的类的所有公共字段（public fields）的数组，包括从父类继承的字段
- Field[] getDeclaredFields()：返回一个包含Class对象所表示的类的所有声明的字段（declared fields）的数组，包括私有、保护和公共字段，但不包括从父类继承的字段
- Field getField(String name)：返回指定名称的公共字段；可以通过字段名（字符串类型）查询字段
- Field getDeclaredField(String name)：返回指定名称的字段；可以通过字段名（字符串类型）查询该类声明的字段

现在我有如下变量：

~~~ java
public String id;  //从父类中继承

private String name;

protected int age;

public int sex;
~~~

~~~ java
Field[] fields = clazz.getFields();

Field[] declaredFields = clazz.getDeclaredFields();

Field name = clazz.getDeclaredField("name");
~~~

将上面三个对象打印后结果如下：

~~~ java
public int reflect.MyClass.sex
public java.lang.String reflect.MyClassFather.id
------------------------------------------
private java.lang.String reflect.MyClass.name
protected int reflect.MyClass.age
public int reflect.MyClass.sex
------------------------------------------
private java.lang.String reflect.MyClass.name
~~~

同样，field对象也可以通过` getModifiers()`等方法来获取权限修饰符，和获取构造器的权限修饰符的方式是完全一样的。通过` field.getType()`可以获取变量的数据类型

如果说你想获取这个变量的值，而值是对象所持有的，所以你需要先创建一个对象，并且如果权限不是共有的你仍需使用` field.setAccessibe(true)`来临时暴力反射，然后再使用` field.get(对象)`来获取具体的值

### 获取成员方法

那其实找规律也能找出来了，获取成员方法的方式肯定也大差不差，不废话直接列出来：

- Method[] getMethod()：返回Class对象所表示的类的所有公共方法，包括从父类继承的公共方法
- Method[] getDeclaredMethod()：返回Class对象所表示的类的所有声明的方法，包括私有、protected和公共方法，但不包括从父类继承的方法
- Method getMethod(String name,Class<?>... parameterTypes)：返回指定名称和参数类型的公共方法
- Method getDeclareMethod(String name,Class<?>... parameterTypes)：返回指定名称和参数类型的方法，包括私有方法

类中有如下方法：

~~~ java
public void display() {
    System.out.println("Hello");
}
private String greet(String greeting) {
    return greeting + ", " + name + "!";
}
~~~

~~~ java
Method[] publicMethods = clazz.getMethods();

Method[] declaredMethods = clazz.getDeclaredMethods();

Method greetMethod = clazz.getDeclaredMethod("greet",String.class);
~~~

注意在获取一个方式时，参数不仅仅只有方法名，同时还要带上入参的类型。这主要是避免方法重载造成的误会，那么打印上面的对象后结果如下：

~~~ 
public void reflect.MyClass.display()
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public java.lang.String java.lang.Object.toString()
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
---------------------------------------------------------------
private java.lang.String reflect.MyClass.greet(java.lang.String)
public void reflect.MyClass.display()
---------------------------------------------------------------
private java.lang.String reflect.MyClass.greet(java.lang.String)
~~~

欸？为什么打印了这么多？但是看过所有类的父类——Object的源码的同学应该很快就能发现其实多出来的方法都是Object中的公有方法，开始我们也说过使用不带Declared的方式去获取成员方法时会返回父类的公有成员方法

拿到这个method对象后，我们可以获取方法的修饰符：` method.getModifiers()`、获取方法的名字：` method.getName()`、获取方法的形参：` method.getParameters()`、获取方法抛出的异常：` method.getExceptionTypes()`以及使用对象来调用方法：`method.invoke(对象,参数)`，如果方法为非公有的仍需` method.setAccessible(true)`来取消权限校验

## 代理

​说了反射那就不得说一说代理。代理（Proxy）是一种设计模式，通过引入一个代理对象来控制对目标对象的访问。代理对象在客户端和真实对象之间充当中介，用户通过代理对象与真实对象进行交互。代理模式可以提供额外的功能，例如权限控制、日志记录、延迟初始化、性能监测等。这个概念听起来和AOP很像对不对，别急后面我们会提到，这篇文章标题之所以叫“从反射出发”就是因为反射引出了很多其它的概念。

​	回归正题，代理是分为静态代理和动态代理的，里面只有动态代理是依赖于反射的，但我们还是两种都讲一讲

### 静态代理

静态代理有以下几个特点：

- **提前定义**：静态代理的代理类在编译时就已经定义好，代码中需要手动创建代理类
- **代码冗余**：对于每一个接口的具体实现类，都需要创建一个对应的代理类。这可能导致代码重复，使得维护变得困难
- **无反射**：静态代理不依赖于Java的反射机制，因此在性能上可能比动态代理更高效
- **简单明确**：静态代理的代理关系在编译时已确定，逻辑简单明了，便于理解

知道了其定义后，我们可以来简单的实现一下。首先定义一个接口：

~~~ java
public interface UserService {
    void addUser(String user);
    void deleteUser(String user);
}
~~~

然后定义一个实现接口的目标类：

~~~ java
public class UserServiceImpl implements UserService {
    @Override
    public void addUser(String user) {
        System.out.println("添加用户: " + user);
    }

    @Override
    public void deleteUser(String user) {
        System.out.println("删除用户: " + user);
    }
}
~~~

创建一个代理类：

~~~ java
public class UserServiceProxy implements UserService {
    private UserService userService;

    public UserServiceProxy(UserService userService) {
        this.userService = userService;
    }

    @Override
    public void addUser(String user) {
        System.out.println("日志：开始添加用户");
        userService.addUser(user); // 调用目标方法
        System.out.println("日志：结束添加用户");
    }

    @Override
    public void deleteUser(String user) {
        System.out.println("日志：开始删除用户");
        userService.deleteUser(user); // 调用目标方法
        System.out.println("日志：结束删除用户");
    }
}
~~~

测试：

~~~ java
public class StaticProxyDemo {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        UserService proxy = new UserServiceProxy(userService);

        proxy.addUser("张三");
        proxy.deleteUser("李四");
    }
}
~~~

![image-20240920203046898](/images/posts/Java/Reflect/image-20240920203046898.png)

***总结***：静态代理是简单易懂的一种代理实现，适用于那些对代理的行为有清晰且固定要求的场景。然而，由于每个代理类都需要手动编写，代码的可维护性和可扩展性受到影响。在实际应用中，通常会根据需求选择适合的代理模式，静态代理在某些简单场景下仍然非常有效。对于需要更高灵活性和可扩展性的应用，动态代理可能是更好的选择。

### 动态代理

静态代理这么呆，那动态代理就一定很灵活了！在Java中，动态代理可以通过两种主要方式实现：JDK动态代理和CGLIB动态代理，我们来看一看两种实现方式有什么不同

#### jdk动态代理

jdk动态代理有以下特点：

- **基于接口**：JDK动态代理要求目标类实现一个或多个接口。代理类在运行时动态生成，并实现了这些接口
- **使用Java反射机制**：通过java.lang.reflect.Proxy类和InvocationHandler接口来实现动态代理
- **简单高效**：因其依赖于Java内建的反射，通常实现简单，性能较高

和静态代理一样，我们先定义一个接口和一个实现类：

~~~ java
public interface HelloService {
    void sayHello(String name);
}

public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello(String name) {
        System.out.println("Hello, " + name);
    }
}
~~~

上面说过java中主要是通过Proxy类和InvocationHandler接口来实现动态代理的，那么这两到底是何方神圣呢？

InvocationHandler接口中只有一个方法：

~~~ java
Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
~~~

- Object proxy：代理对象的引用，这是代理类的实例
- Method method：被调用的方法的Method对象，包含方法的信息，包括名称、参数类型等
- Object[] args：调用方法时传递的参数

返回值即是被调用方法的返回值。而Proxy中主要的静态方法为：

~~~ java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
~~~

- ClassLoader loader：定义了由哪个类加载器加载代理对象的接口
- Class<?>[] interfaces：一个Class对象的数组，表示要实现的接口。这些接口定义了代理对象所应该遵循的行为
- InvocationHandler h：指定了处理代理对象的方法调用的处理器

想一想我们现在的目的是什么——给类HelloServiceImp扩展功能，那比如我们现在想扩展的功能就是在sayHello这个方法执行前和执行后都打印一段日志，那这个具体功能我们是不是就要找到一个合适的处理器？所以这就是InvocationHandler的作用：

~~~ java
class MyInvocationHandler implements InvocationHandler {
    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = method.invoke(target, args); // 调用目标方法
        System.out.println("After method: " + method.getName());
        return result;
    }
}
~~~

我们可以先关注重写的invoke方法，其实很简单就是加上了我们想要的功能。现在这个附加功能的处理器已经有了，可以看出这就是newProxyInstance中的第三个参数，那我们还缺一个载体来运行这个处理器，我们可以把它叫做代理：

~~~ java
public class JdkDynamicProxyDemo {
    public static void main(String[] args) {
        HelloService target = new HelloServiceImpl(); // 目标对象
        HelloService proxy = (HelloService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new MyInvocationHandler(target)
        );

        proxy.sayHello("张三"); // 调用代理的方法
    }
}
~~~

我们得到了这样一个代理对象proxy，那么也就可以用代理来调用被代理的方法，结果如下：

![image-20240920212628756](/images/posts/Java/reflect/image-20240920212628756.png)

这样一个简单的基于jdk的动态代理就完成了。那么这里还可以思考以下两个问题：

1. 代理类是否可以强转成目标类？
2. 目标类是否可以是final类型?

这两个问题的答案是false、true。因为无论是我们的目标类还是代理类，他们都实现了同一个接口，就相当于他们之间是兄弟关系，兄弟关系不能互相转换，就像猫和狗都实现了动物接口但不能转换。而两者之间没有继承关系，所以就可以为final类型。关于这里为什么要强调兄弟关系，是因为接下来要讲到的cglib的实现方式有所不同

#### cglib动态代理

jdk动态代理有一个问题就是只能代理实现了接口的类，那代理一个没有实现接口的类该如何做到呢？cglib动态代理就可以避免这个问题。cglib动态代理有以下特点：

- **基于子类**：CGLIB动态代理通过继承目标类来实现代理，即使目标类没有实现接口也能使用CGLIB来进行代理，因此可以代理类方法。
- **字节码生成**：CGLIB使用ASM字节码生成库，通过动态生成目标类的子类来实现代理。
- **相对较重**：因为它运行时生成字节码，CGLIB的性能相对JDK动态代理稍低，但通常仍然较快。

Spring中的AOP模块用的即是cglib代理。想用cglib代理之前首先得引入依赖：

~~~ xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
~~~

首先定义一个没有实现接口的目标类：

~~~ java
public class HelloService {
    public void sayHello(String name) {
        System.out.println("Hello, " + name);
    }
}
~~~

cglib动态代理也有自己的两大核心利器分别是Enhancer类和MethodInterceptor接口

~~~ java
class MyMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args, 
                            MethodProxy proxy) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = proxy.invokeSuper(obj, args); // 调用目标方法
        System.out.println("After method: " + method.getName());
        return result;
    }
}
~~~

~~~ java
public class CglibDynamicProxyDemo {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(HelloService.class);
        enhancer.setCallback(new MyMethodInterceptor()); // 设置拦截器

        HelloService proxy = (HelloService) enhancer.create(); // 创建代理实例
        proxy.sayHello("张三"); // 调用代理的方法
    }
}
~~~

由于我现在使用的是java1.8，而使用cglib代理需要java9以上的版本，所以验证环节就先欠着,有时间再来完善:smile:

## AOP

AOP 是一种编程范式，它通过将关注点（如日志、事务管理、安全性等）模块化，从而提供更好的代码组织和可维护性。AOP 的核心理念是把横切关注点（cross-cutting concerns）与业务逻辑分离，允许你在不修改核心业务逻辑的情况下插入这种横切关注点。

前面也提到了反射和AOP很像，Spring AOP中是用到了动态代理，而动态代理又是基于反射。除了Spring AOP外还有AspectJ AOP，后者不依赖于代理而依赖于字节码操作。Spring AOP中已经集成了AspectJ，Sring相对于AspectJ要简单一些。这里只对Spring AOP的概念和注解进行简单的介绍，可以在另一篇文章中看详细使用。

Spring AOP的核心概念：

1.  **切面（Aspect）**：切面是一个包含了横切关注点的类，通常由切点和通知组成。
2.  **切点（Pointcut）**：切点是定义某个具体位置（如方法调用或类实例化）的一种表达方式，它指定了在哪些地方应用某个通知。
3.  **通知（Advice）**：通知是切面中执行的动作，可以在切点之前、之后或抛出异常时执行。Spring支持多种类型的通知，包括前置通知、后置通知、环绕通知等。
4.  **连接点（Joinpoint）**：连接点是指程序执行的某一个点，例如方法调用、对象实例化等。在Spring中，连接点代表的方法执行。
5.  **引织（Weaving）**：引织是将切面与其目标对象结合的过程，可以在编译时、类加载时或运行时进行。

Spring AOP提供了一组注解，用于简化切面编程的配置和实现。这些注解使得定义切面、切点和通知变得更加直观和易于使用：

1. **@Aspect**:标记一个类为切面。使用这个注解的类包含了切点和通知的定义
2. **@Pointcut**:用于定义切点的注解。切点可以是其他通知的引用，通常会和`@Before`、`@After`等通知一起使用
3. **@Before**:前置通知，表示在目标方法执行之前执行的逻辑。可以用来实现日志记录、权限检查等功能
4. **@After**:后置通知，表示在目标方法执行之后执行的逻辑。无论目标方法是否抛出异常，这段代码都会执行
5. **@Around**:环绕通知，允许在目标方法执行前后执行额外的逻辑。它是最强大的通知类型，可以控制方法的执行
6. **@AfterReturning**：在目标方法成功执行后执行的通知，可以获取目标方法的返回值，常用于日志记录或结果处理
7. **@AfterThrowing**：在目标方法抛出异常后执行的通知，可以获取异常信息，常用于异常处理或日志记录

以上注解提供了实现AOP的基础设施，允许开发者以声明式的方式定义切面、切点和通知。使用这些注解，开发者可以专注于业务逻辑的实现，而将横切关注点（如日志、安全、事务等）分离到切面中，从而提高代码的可维护性和可读性。
