---
title: YAML语言
date: 2016/10/29
tag:
    - YAML
    - 标记语言
---

编程中经常会写配置文件，本人比较喜欢使用 JSON，在我以往使用的语言都对 JSON 有良好的支持。
当我使用 [Hexo](https://hexo.io/) 来构建这个博客时， 配置 Hexo 时发现她使用了 YAML 来编写配置文件。google之，发现 YAML 包含了 JSON 的优点（部分吧），还能存储复杂的数据结构。

为什么没提 ini xml 来维护配置文件。编程从PHP开始，对xml支持不友好，解析xml比json麻烦得多。ini使用较少。

#### 简介

YAML ： YAML Ain't Markup Language
YAML 是专门用来写配置文件的语言。

##### 语法规则

* 大小写敏感
* 缩进：

    1. 使用缩进表示层级关系
    2. 缩进时不允许使用Tab键，只允许使用空格。
    3. 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

* 注释：

    `#`表示注释，从这个字符一直到行尾，都会被解析器忽略。

#### 支持的数据结构

#####  对象

键值对的集合，又称为映射（mapping） / 哈希（hashes） / 字典（dictionary）
使用使用冒号结构表示：
```yml
animal: cat
```
JavaScript表示：
```js
{ animal: 'cat' }
```
Yaml 也允许将所有键值对写成一个行内对象。

```yml
hash: { name: Steve, foo: bar }
```
JavaScript：
```js
{ hash: { name: 'Steve', foo: 'bar' } }
```

##### 数组

数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
一组连词线开头的行，构成一个数组。

```yml
- Cat
- Dog
- Goldfish
```

JavaScript :

```js
[ 'Cat', 'Dog', 'Goldfish' ]
```
数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。

```yml
-
 - Cat
 - Dog
 - Goldfish
```
JavaScript :

```js
[ [ 'Cat', 'Dog', 'Goldfish' ] ]
```

当然数组也可以采用行内表示法。
```yml
animal : [cat, dog, Goldfish]
```

JavaScript :
```yml
{ animal: [ 'Cat', 'Dog', 'Goldfish' ] }
```

##### 复合结构

对象和数组可以结合使用形成复合结构。

```yml
languages:
 - Ruby
 - Perl
 - Python
websites:
 YAML: yaml.org
 Ruby: ruby-lang.org
 Python: python.org
 Perl: use.perl.org
```

JavaScript:

```js
{
    languages: [ 'Ruby', 'Perl', 'Python' ],
    websites: {
        YAML: 'yaml.org',
        Ruby: 'ruby-lang.org',
        Python: 'python.org',
        Perl: 'use.perl.org' }
}
```

##### 配置纯量

纯量是最基本的、不可再分的值。


###### 布尔值
布尔值用true和false表示。
```yml
isSet: true
```
###### 整数 & 浮点数

数值直接以字面量的形式表示
*  整数
    ```yml
    number: 12
    ```
* 浮点数

    ```yml
    number: 12.30
    ```

###### NULL
NULL比较特殊，NULL使用 `~` 来表示

```yml
# 注意区分 ~ 和 `
isSet: ~
```
###### 字符串

```yml

```

* 时间
* 日期
