---
layout: post
title: Java 反射基础
date: 2018-01-19 10:01:19
categories: Java
tags: Java反射
author: renshuo
mathjax: true
---

* content
{:toc}
能够分析类能力的程序称为反射(`reflective`)，反射机制能够动态的操纵java代码，在**运行中**分析类能力和查看对象，并且可以在**运行中**操作类或对象的内部属性或方法。反射大量地应用于 `JavaBeans` 中。


<!--more-->

官方对 `Reflection` 的介绍：  [*java8 ：Java Reflection API*](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html)

> Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions. 
>
> The API accommodates applications that need access to either the public members of a target object (based on its runtime class) or the members declared by a given class. It also allows programs to suppress default reflective access control.

有关`java`反射的代码在 `java.lang.reflect` 包中。 [java8：Package java.lang.reflect](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html)

## Class 类

> java8 API：java.lang.Class<T>
>
> Instances of the class Class represent classes and interfaces in a running Java application. An enum is a kind of class and an annotation is a kind of interface. Every array also belongs to a class that is reflected as a Class object that is shared by all arrays with the same element type and number of dimensions. The primitive Java types (boolean, byte, char, short, int, long, float, and double), and the keyword void are also represented as Class objects.
> Class has no public constructor. Instead Class objects are constructed automatically by the Java Virtual Machine as classes are loaded and by calls to the defineClass method in the class loader.

谷歌翻译：

> Class类的实例表示正在运行的Java应用程序中的类和接口。 一个枚举是一种类，一个注释是一种接口。 每个数组也都属于一个被反映为一个Class对象的类，它被具有相同元素类型和维数的所有数组共享。 原始的Java类型（布尔型，字节型，字符型，短型，int型，long型，float型和double型）以及关键字void也被表示为Class对象。
>
> Class类没有公共构造函数。 相反，Class对象是由JVM自动构造的，因为在加载类时以及通过调用类加载器中的defineClass方法。

也就是说，Class类保存了声明的对象所在的类的信息。当装载类时，Class类型的对象由JVM自动创建。所以通过Class类和反射库中的类结合使用，可以实现java的反射机制。

## 常见用法

### 获取一个Class类对象

1. 通过字符串

``` java
String name = "java.util.Date";
Class cl = Class.forName(name);

// 获取父类
Class supCl = cl.getSuperclass();
```

2. 通过对象

``` java
Date d = new Date();
Class cl = d.getClass();
```

3. 基本类型

``` java
Class s = String.class;
Class i = int.class;
```

### 创建实例

``` java
String name = "java.util.Date";
Class cl = Class.forName(name);

Object obj = cl.newInstance();
// 需要强制转换
Date date = (Date) cl.newInstance();
```

在使用Class类的 `newInstance()` 方法创建对象实例时，调用的是对象类的无参数构造函数。如果没有无参数构造函数，会产生异常。所以推荐使用`Constructor.newInstance`方法。

### 分析类

`Class`类中的一些方法可以获取目标类的信息，包括构造器、方法、属性域、注解和接口等内容，本文只做简单的介绍，详细信息请看 `API` 文档。

#### 构造器(Constructor)

[`java.lang.reflect.Constructor`](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html) 反射库中的构造函数类。通过`Class`类中的方法，可以获得 `Constructor` 类的对象。

| 方法名                                      | 返回类型               | 说明                    |
| ---------------------------------------- | ------------------ | --------------------- |
| [`getConstructors()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructors--) | `Constructor<?>[]` | 返回所有`public`构造器，包括父类的 |
| [`getConstructor(Class[])`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructor-java.lang.Class...-) | `Constructor<T>`   | 根据参数返回 `public` 构造器   |
| [`getDeclaredConstructors()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructors--) | `Constructor<?>[]` | 返回本类的所有构造器            |
| [`getDeclaredConstructor(Class<?>... parameterTypes)`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructor-java.lang.Class...-) | `Constructor<T>`   | 根据参数查找本类的构造器          |

#### 方法(Method)

[`java.lang.reflect.Method`](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html) 反射库中的方法类。通过 `Class` 类中的方法，可以获得 `Method` 类的对象。

| 方法名                                      | 返回类型       | 说明                             |
| ---------------------------------------- | ---------- | ------------------------------ |
| [`getMethods()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethods--) | `Method[]` | 返回所有的`public`方法，包括从父类继承的和接口实现的 |
| [`getMethod(String, Class[])`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethod-java.lang.String-java.lang.Class...-) | `Method`   | 通过名称和参数获取具体的`public`方法         |
| [`getDeclaredMethods()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethods--) | `Method[]` | 获取所有的本类实现的方法，包括接口的不包括继承的       |
| [`getDeclaredMethod(String, Class[])`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethod-java.lang.String-java.lang.Class...-) | `Method`   | 在上面的基础上，根据名称和参数获取具体的方法         |

#### 域(Field)

[`java.lang.reflect.Field`](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html) 反射库中的类的域类，通过 `Class` 类中的方法，可以获得 `Field` 类的对象。

| 方法名                                      | 返回类型      | 说明                        |
| ---------------------------------------- | --------- | ------------------------- |
| [`getFields()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getFields--) | `Field[]` | 返回所有`public`声明的域，包括父类和接口的 |
| [`getField(String)`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getField-java.lang.String-) | `Field`   | 根据名称获取方法                  |
| [`Class.getDeclaredFields()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredFields--) | `Field[]` | 返回所有域，包括接口的不包括继承的         |
| [`Class.getDeclaredField(String)`](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredField-java.lang.String-) | `Field`   | 在上面的基础上，根据名称获取域           |

### 例子

``` java
package reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.Scanner;

/**
 * java 反射基本用法
 * Created by renshuo on 2018/1/19.
 */
public class Reflection {
    public static void main(String[] args) {
        String name;
        Scanner in = new Scanner(System.in);
        System.out.println("输入要分析的类，e.g. java.util.Date");
        name = in.next();
        try {
            Class cl = Class.forName(name);
            // 获取修饰词
            String modifiers = Modifier.toString(cl.getModifiers());
            if (modifiers.length() > 0) {
                System.out.println("modifiers: " + modifiers);
            }
            System.out.println("Class:" + cl.getName());
            // 父类
            Class superCl = cl.getSuperclass();
            if (superCl != null && superCl != Object.class) {
                System.out.println("extends " + superCl.getName());
            }
            System.out.println("{");
            printFields(cl);
            printConstructors(cl);
            printMethods(cl);
            System.out.print("}");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    /**
     * 分析类中所有的属性变量
     */
    private static void printFields(Class cl) {
        Field[] fields = cl.getFields();
        for (Field f : fields) {
            System.out.println("    " + f);
        }
    }
    /**
     * 分析构造函数
     */
    private static void printConstructors(Class cl) {
        Constructor[] constructors = cl.getConstructors();
        for (Constructor c : constructors) {
            System.out.println("    " + c);
        }
    }
    /**
     * 分析类方法
     */
    private static void printMethods(Class cl) {
        Method[] methods = cl.getMethods();
        for (Method m : methods) {
            System.out.println("    " + m);
        }
    }
}
```

## invoke方法

`java.lang.reflect.Method` 有一个 `invoke` 方法，可以让我们通过它来调用任意方法。如果返回类型是基本类型，`invoke`方法会返回其包装器类型。使用时可能需要强制类型装换。

``` java 
public Object invoke(Object obj, // 隐式参数，静态方法可以忽略该参数，传入的是对象实例
                     Object... args) // 函数方法参数
              throws IllegalAccessException,
                     IllegalArgumentException,
                     InvocationTargetException
```

简单使用说明

``` java
package reflection;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * Created by renshuo on 2018/1/20.
 */
public class invokeTest {
    public static void main(String[] args) throws ClassNotFoundException,
            NoSuchMethodException, InvocationTargetException,
            IllegalAccessException, InstantiationException {
        String name = "reflection.methodClass";
        Class cl = Class.forName(name);
        // 调用无参数构造器创建实例
        Object obj = cl.newInstance();

        Method methodAdd = cl.getMethod("add", int.class, int.class);
        int c = (int) methodAdd.invoke(obj, 1, 2);
        System.out.println(c);
    }
}
class methodClass {
    public final int f = 3;
    public int add(int a, int b) {
        return a + b;
    }
    public int sub(int a, int b) {
        return a + b;
    }
}
```

对于 `private` 等默认没有权限访问的方法，需要使用 `setAccessible()` 方法获取权限

``` java 
Method m = obj.getClass().getDeclaredMethod(methodName);
m.setAccessible(true);
m.invoke(obj, args);
```

