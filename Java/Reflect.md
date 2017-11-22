---
title: Java 反射机制
date: '2016/10/8 22:22:54'
tag:
  - java
urlname: using-Java-Reflection
---



要让Java程序能够运行，就得让Java类被Java虚拟机加载。Java类如果不被Java虚拟机加载就不能正常运行。正常情况下，我们运行的所有的程序在编译期时候就已经把那个类被加载了。

Java编译时并不确定是哪个类被加载了，而是在程序运行的时候才自审、探知、加载所需要的类。在编译期时使用并不知道的类。这样的编译特点就是java反射。
<!--more-->
# Java反射的作用

假如有两个程序员，一个程序员在写程序的时需要使用第二个程序员所写的类，但第二个程序员并没完成他所写的类。那么第一个程序员的代码是不能通过编译的。此时，利用Java反射的机制，就可以让第一个程序员在没有得到第二个程序员所写的类的时候，来完成自身代码的编译。

Java的反射机制它知道类的基本结构，这种对Java类结构探知的能力，我们称为Java类的"自审"。如IDE中，在对象后按下`.`，编译工具就会自动的把该对象能够使用的所有的方法和属性全部都列出来，供用户进行选择。这就是利用了Java反射的原理，是对我们创建对象的探知、自审。

# Class 类

要正确使用Java反射机制就得使用java.lang.Class这个类，即要使用Java反射机制的类要继承于java.lang.Class这个类。它是Java反射机制的起源。当一个类被加载以后，Java虚拟机就会自动产生一个Class对象。通过这个Class对象我们就能获得加载到虚拟机当中这个Class对象对应的方法、成员以及构造方法的声明和定义等信息。

# 反射功能

通过反射机制可以获得

1. Class 对象
2. 类名
3. 修饰符
4. 包信息
5. 父类
6. 实现的接口
7. 构造器
8. 方法
9. 变量
10. 注解

Java反射机制使用java.lang.Class这个类，可以在JavaDoc中的[java.lang.Class](http://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)查看到更多的描述。

## 获取Class对象

编译期知道一个类的名字的话，那么你可以使用如下的方式获取一个类的 Class 对象。

```java
Class classA = ClassName.class;
```

如果你在编译期不知道类的名字，但是你可以在运行期获得到类名的字符串,那么你则可以这么做来获取 Class 对象:

```java
String className = "GetClassName" ; // 在运行期获取的类名字符串
Class classB = Class.forName(className);
```

在使用 Class.forName() 方法时，你必须提供一个类的全名，全名即是包名加上类名。比如MyObject类在com.example.myapp包中，那么他的全名是com.example.myapp.MyObject。如果在调用Class.forName()方法时，没有在编译路径下(classpath)找到对应的类，那么将会抛出ClassNotFoundException。

## 获取类名

java.lang.Class提供了两个获取类名的方法。

```java
Class classA = ClassName.class;

// 通过 getName() 方法返回类的全名 包名+类名
String className = classA.getName();
// 使用getSimpleName() 仅仅可以得到类名
String simpleClassName = classA.getSimpleName();
```

## 获取修饰符

可以通过 Class 对象来访问一个类的修饰符， 即public,private,static 等等的关键字，修饰符都被包装成一个int类型的数字，这样每个修饰符都是一个位标识(flag bit)，这个位标识可以设置和清除修饰符的类型。 可以使用 java.lang.reflect.Modifier 类中的方法来检查修饰符的类型：

```java
Class classA = ClassName.class;
int modifiers = classA.getModifiers();

Modifier.isAbstract(int modifiers);
Modifier.isFinal(int modifiers);
Modifier.isInterface(int modifiers);
Modifier.isNative(int modifiers);
Modifier.isPrivate(int modifiers);
Modifier.isProtected(int modifiers);
Modifier.isPublic(int modifiers);
Modifier.isStatic(int modifiers);
Modifier.isStrict(int modifiers);
Modifier.isSynchronized(int modifiers);
Modifier.isTransient(int modifiers);
Modifier.isVolatile(int modifiers);
```

## 获取包信息

通过 Package 对象你可以获取包的相关信息，比如包名，获取更多的 Package 类信息可以阅读 [java.lang.Package](http://docs.oracle.com/javase/8/docs/api/java/lang/Package.html)。

```java
Class classA = ClassName.class;
Package package = classA.getPackage();
```

## 获取父类

通过 Class 对象你可以访问类的父类，可以看到 superclass 对象也是一个 Class 类的实例，所以你可以继续在这个对象上进行反射操作。

```java
Class classA = ClassName.class;
Class superClass = classA.getSuperClass();
```

## 获取被实现的接口

可以获取指定类所实现的接口集合，由于一个类可以实现多个接口，因此 getInterfaces(); 方法返回一个 Class 数组，在 Java 中接口同样有对应的 Class 对象。 注意：getInterfaces() 方法仅仅只返回当前类所实现的接口。当前类的父类如果实现了接口，这些接口是不会在返回的 Class 集合中的，尽管实际上当前类其实已经实现了父类接口。

```java
Class  classA = ClassName.class;
Class[] interfaces = classA.getInterfaces();
```

## 构造器

你可以通过如下方式访问一个类的构造方法，更多有关 Constructor 的信息可以访问 [Constructor](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html)。

```java
Constructor[] constructors = aClass.getConstructors();
```

## 方法

你可以通过如下方式访问一个类的所有方法：更多有关 Method 的信息可以 [Method](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html)

```java
Method[] method = aClass.getMethods();
```

## 变量

你可以通过如下方式访问一个类的成员变量，更多有关 Field 的信息可以访问[Field](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html)。

```java
Field[] method = aClass.getFields();
```

## 注解

你可以通过如下方式访问一个类的注解：更多有关 Annotation 的信息可以 [Annotation](http://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Annotation.html)

```java
Annotation[] annotations = aClass.getAnnotations();
```
