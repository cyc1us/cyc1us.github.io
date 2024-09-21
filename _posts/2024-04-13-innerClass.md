---
layout: post
title: "丰富的Java内部类"
date: 2024-04-13 
description: "Java内部类详解"
tag: Java
--- 
内部类在平时用的很多，但可能都是用了就用了，甚至什么时候用的都不知道，没有去深究其原理。每每想去深究其原理又被各种复杂的描述、繁多的分类所吓退，我也是被这些复杂的概念折磨得不堪其苦，所以这次好好的写一个对内部类的总结，也算是逼自己去统一的学习一下内部类

## 为什么使用内部类？

内部类最吸引人的原因是：每个内部类都能独立地继承一个接口或实现类，所以无论外部类是否已经继承了某个（接口的）实现，对于内部类都没有影响，使用内部类最大的优点就在于它能够非常好的解决多重继承的问题,使用内部类还能够为我们带来如下特性:

1. 内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立
2. 创建内部类对象的时刻并不依赖于外围类对象的创建
3. 内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体
4. 内部类提供了更好的封装，除了该外围类，其他类都不能访问

下面这段代码展示了上述特性：

~~~ java
interface Runnable {
    void run();
}
class Outer {
    private int outerVar = 10;

    class Inner implements Runnable {
        @Override
        public void run() {
            System.out.println("Running with outer variable: " + outerVar);
        }
    }

    public Runnable getRunnable() {
        return new Inner();
    }
}
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer();
        Runnable runnable = outer.getRunnable();
        runnable.run();
    }
}
~~~

在这个例子中，Outer类本身没有实现Runnable接口，但通过内部类Inner，我们实现了Runnable接口的功能。这使得Outer类可以提供一个Runnable对象，而不需要自己实现Runnable接口

## 内部类的分类

### 成员内部类

```java
class Outer {
    private int outerVar = 10;

    class Inner {
        void display() {
            System.out.println("Outer variable: " + outerVar);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
        inner.display();
    }
}

```

1. Inner 类定义在Outer类的内部，相当于Outer类的一个成员变量的位置，Inner 类可以使用任意访问控制符，就如同类中的一个成员变量
2. Inner 类中display()方法可以直接访问Outer类中的数据，而不受访问控制符的影响，如直接访问Outer类中的私有属性outerVar
3. 定义了成员内部类后，必须使用外部类对象来创建内部类对象，而不能直接去new一个内部类对象
4. 编译上面的程序后，会发现产生了两个 .class 文件: Outer.class,Outer$Inner.class{}
   ![image-20240913095853320](/images/posts/Java/innerClass/image-20240913095853320.png)
5. 成员内部类中不能存在任何 static 的变量和方法,可以定义常量:

   - 非静态内部类要依赖于外部类的实例,而静态变量和方法是不依赖于实例对象的,仅与类相关,简而言之:在加载静态域时,根本没有外部类,所在在非静态内部类中不能定义静态域或方法,编译不通过

   - 常量是在编译期就确定的,放到常量池

此外，外部类不能直接使用内部类的成员和方法，需要先创建内部类的对象，然后通过内部类的对象来访问其成员变量和方法;如果外部类和内部类具有相同的成员变量或方法，内部类默认访问自己的成员变量或方法，如果要访问外部类的成员变量，可以使用 this 关键字
### 静态内部类

带静态两个字，那很明显是用static 修饰的内部类，有以下特点：

1. 静态内部类不能直接访问外部类的非静态成员，但可以通过new 外部类().成员 的方式访问 
2. 如果外部类的静态成员与内部类的成员名称相同，可通过“类名.静态成员”访问外部类的静态成员；如果外部类的静态成员与内部类的成员名称不相同，则可通过“成员名”直接调用外部类的静态成员
3. 创建静态内部类的对象时，不需要外部类的对象，可以直接创建

```java
class Outer {
    private static int outerVar = 10;

    static class Inner {
        void display() {
            System.out.println("Outer variable: " + outerVar);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Outer.Inner inner = new Outer.Inner();
        inner.display();
    }
}
```

### 局部内部类 

局部内部类的作用域仅限于方法内，方法外部无法访问该内部类，特点如下：

1. 局部内部类就像是方法里面的一个局部变量一样，不能用访问修饰符以及static 修饰符的，且只能在方法内部使用

2. 可以访问外部类的成员和方法的局部变量（必须是final或实际上是final的变量），这个规则可以从以下几点来理解：

   - 当方法被调用时，局部变量在方法的栈帧中被创建；当方法执行完毕后，局部变量的生命周期结束，它们会被销毁
   - 内部类对象的生命周期可能比方法的执行时间更长；即使方法执行完毕，内部类对象可能仍然存在，直到没有被引用时才会被垃圾回收
   - 如果内部类对象在方法执行完毕后仍然存在，并且尝试访问方法中的局部变量，这些局部变量已经不存在了，这会导致内部类访问一个不存在的变量，从而引发错误

   为了解决上述问题，Java要求局部内部类只能访问方法中定义的final类型的局部变量。final 修饰符确保变量的引用不会改变，并且编译器会持续维护这个变量在回调方法中的生命周期
   ~~~ java
   public class Outer {
   
       void someMethod() {
           final int localVar = 20; // 局部变量必须是 final
   
           class Inner {
               void display() {
                   System.out.println("Local variable: " + localVar);
               }
           }
   
           Inner inner = new Inner();
           inner.display();
       }
   
       public static void main(String[] args) {
           Outer outer = new Outer();
           outer.someMethod();
       }
   }
   ~~~

3. 然而，从 JDK 8 开始，Java 引入了**Effectively final**[^1]的概念，这使得方法内部类和匿名内部类可以访问方法中的局部变量，而不需要显式地将这些变量声明为 final
   
   ~~~ java
   public class Outer {
       void someMethod() {
           int localVar = 20; // 不需要显式声明为 final
   
           class Inner {
               void display() {
                   System.out.println("Local variable: " + localVar);
               }
           }
   
           Inner inner = new Inner();
           inner.display();
       }
   }
   ~~~
   
   将上述代码进行反编译后可以发现内部类引用外部的局部变量是final修饰的
   ![image-20240913095821312](/images/posts/Java/innerClass/image-20240913095821312.png)

### 匿名内部类

1. 匿名内部类是直接使用new来生成一个对象的引用
2. 对于匿名内部类的使用它是存在一个缺陷的，就是它仅能被使用一次，创建匿名内部类时它会立即创建一个该类的实例，该类的定义会立即消失，所以匿名内部类不能够被重复使用
3. 使用匿名内部类时，我们必须是继承一个类或者实现一个接口，但是两者不可兼得，同时也只能继承一个类或者实现一个接口
4. 匿名内部类中不能定义构造函数,匿名内部类中不能存在任何的静态成员变量和静态方法
5. 匿名内部类中不能存在任何的静态成员变量和静态方法,匿名内部类不能是抽象的,它必须要实现继承的类或者实现的接口的所有抽象方法

```java
interface Greeting {
    void greet();
}

public class Main {
    public static void main(String[] args) {
        Greeting greeting = new Greeting() {
            @Override
            public void greet() {
                System.out.println("Hello, World!");
            }
        };

        greeting.greet();
    }
}
```

## 常见使用

其实大部分人可能都是知道有内部类这个东西的，但一问到平时在哪里有使用过就发懵了。其实并不是你没有使用过，相反用到的地方还很多，只是平时忽略了。

### 回调函数
在异步编程中，回调函数是常见的模式。使用内部类可以方便地定义回调函数，Lambda表达式实际上就是对匿名内部类的一种简化书写方式

~~~ java
public class AsyncExample {

    public static void main(String[] args) {
        CompletableFuture.runAsync(() -> {
            System.out.println("Running async task");
        }).thenAccept(result -> {
            System.out.println("Async task completed");
        });
    }
}

~~~

### 过滤器和拦截器
在Web开发中，过滤器和拦截器用于处理请求和响应。使用内部类可以方便地定义这些组件

~~~ java
@WebFilter("/myPath")
public class MyFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Filtering request");
        chain.doFilter(request, response);
    }
}
~~~

### Spring框架中的Bean定义
在Spring框架中，可以使用内部类来定义Bean，特别是在配置类中。

~~~ java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
    private class MyService {
        public void doSomething() {
            System.out.println("Doing something");
        }
    }
}

~~~

### 多线程处理
在多线程编程中，内部类可以用于定义线程类或实现Runnable接口。
~~~ java
public class MultiThreadExample {

    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start();
    }

    private static class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println("Running in a separate thread");
        }
    }
}
~~~

### 实体类

~~~ java
public class User {
    private Long id;

    private String username;

    private String email;

    private Address address;

    public static class Address {

        private String street;

        private String city;

        private String state;

        private String zipCode;
    }
}
~~~

在日常开发中可能会遇到一个类中需要另一个类的属性，为了简化开发可用内部类来定义

***

[^1]:Effectively final是指一个变量虽然没有显式地声明为final，但在整个生命周期中，它的值没有被修改过。换句话说，如果一个变量在声明后没有被重新赋值，那么它就被认为是Effectively final。引入Effectively final的主要目的是为了简化代码，减少不必要的final声明。在实际编程中，很多局部变量在声明后确实不会被修改，但开发者仍然需要显式地声明它们为final，这增加了代码的冗余。
