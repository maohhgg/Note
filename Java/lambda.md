---
title: Lambda表达式简介
date: '2016/10/7 22:22:54'
tag:
  - java
urlname: Lambda
---

Lambda表达式是java8给我们带来的几个重量级新特性之一，借用Lambda表达式，可以让我们的java程序设计更加简洁。Android Studio也会使用lambda表达式来缩略展示代码，本文将简略介绍Lambda表达式。

# Lambda 表达式简介

Lambda 表达式是一种匿名函数(对Java而言这并不完全正确，但现在姑且这么认为)，简单地说，它是没有声明的方法，也即没有访问修饰符、返回值声明和名字。

你可以将其想做一种速记，在你需要使用某个方法的地方写上它。当某个方法只使用一次，而且定义很简短，使用这种速记替代之尤其有效，这样，你就不必在类中费力写声明与方法了。

Java 中的 Lambda 表达式通常使用 (argument) -> (body) 语法书写，例如：

```java
(arg1, arg2...) -> { body }

(type1 arg1, type2 arg2...) -> { body }
```

实例：

```java
(int a, int b) -> {  return a + b; }

() -> System.out.println("Hello World");

(String s) -> { System.out.println(s); }

() -> 42

() -> { return 3.1415 };
```

# Lambda 表达式的结构

- 一个 Lambda 表达式可以没有或多个参数
- 参数的类型既可以明确声明，也可以根据上下文来推断。例如：`(int a)`与`(a)`效果相同
- 所有参数需包含在圆括号内，参数之间用逗号相隔。例如：`(a, b)`或`(int a, int b)`或`(String a, int b, float c)`
- 空圆括号代表参数集为空。例如：`() -> 42`
- 当只有一个参数，且其类型可推导时，圆括号()可省略。例如：`a -> return a*a`
- Lambda 表达式的主体可没有主题或包含多条语句
- 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
- 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

# Lambda 表达式示例

```java
//原来的方法
List<String> list = Arrays.asList("aaa", "bbb", "ccc");
list.sort(new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return Integer.compare(o1.length(), o2.length());
    }
});

//使用Lambda表达式
List<String> list = Arrays.asList("aaa", "bbb", "ccc");
list.sort((o1, o2) -> Integer.compare(o1.length(), o2.length()));
```

```java
//原来的方法
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
for(Integer n: list) {
   System.out.println(n);
}

//使用Lambda表达式
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
list.forEach(n -> System.out.println(n));
```

使用 Lambda 表达式的方法不止一种。我们常用的箭头语法创建 Lambda 表达式， Java 8中还可以使用双冒号(::)操作符。

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
list.forEach(System.out::println);
```

# 函数式接口

java.lang.Runnable 就是一种函数式接口，在 Runnable 接口中只声明了一个方法 void run()，我们使用匿名内部类来实例化函数式接口的对象，有了 Lambda 表达式，这一方式可以得到简化。

每个 Lambda 表达式都能隐式地赋值给函数式接口，例如，我们可以通过 Lambda 表达式创建 Runnable 接口的引用。

```java
// Lambda 表达式
Runnable r = () -> System.out.println("hello world");

//编译器会自动解释这种转化
new Thread(
   () -> System.out.println("hello world")
).start();

//不使用Lambda 表达式
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("hello world");
    }
}).start();
```

像上面示例中Comparator 接口也算是：

```java
// Lambda 表达式
Comparator cmp = (x, y) -> (x < y) ? -1 : ((x > y) ? 1 : 0);

Comparator cmp = (x, y)->{
    return (x < y) ? -1: ((x > y)?1:0);
};

//原来的方法
newComparator(){
    @Override
    publicint compare(Integer x, Integer y){
        return (x< y) ? -1 :((x> y) ? 1 : 0);
    }
};
```
