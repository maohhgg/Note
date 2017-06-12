---
title: YAML语言
date: 2016/10/29
tag:
  - YAML
  - 标记语言
urlname: YAML-language
---

编程中经常会写配置文件，本人比较喜欢使用 JSON，在我以往使用的语言都对 JSON 有良好的支持。 当我使用 [Hexo](https://hexo.io/) 来构建这个博客时， 配置 Hexo 时发现她使用了 YAML 来编写配置文件。google之，发现 YAML 包含了 JSON 的优点（部分吧），还能存储复杂的数据结构。

为什么没提 ini xml 来维护配置文件。编程从PHP开始，对xml支持不友好，解析xml比json麻烦得多。ini使用较少。

# 简介

YAML ： YAML Ain't Markup Language YAML 是专门用来写配置文件的语言。

## 语法规则

- 大小写敏感
- 缩进：

  1. 使用缩进表示层级关系
  2. 缩进时不允许使用Tab键，只允许使用空格。
  3. 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

- 注释：

  `#`表示注释，从这个字符一直到行尾，都会被解析器忽略。

# 支持的数据结构

## 对象

键值对的集合，又称为映射（mapping） / 哈希（hashes） / 字典（dictionary） 使用使用冒号结构表示：

```yml
animal: cat
```

JavaScript表示：

```javascript
{ animal: 'cat' }
```

Yaml 也允许将所有键值对写成一个行内对象。

```yml
hash: { name: Steve, foo: bar }
```

JavaScript：

```javascript
{ hash: { name: 'Steve', foo: 'bar' } }
```

## 数组

数组：一组按次序排列的值，又称为序列（sequence） / 列表（list） 一组连词线开头的行，构成一个数组。

```yml
- Cat
- Dog
- Goldfish
```

JavaScript :

```javascript
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

```javascript
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

## 复合结构

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

```javascript
{
    languages: [ 'Ruby', 'Perl', 'Python' ],
    websites: {
        YAML: 'yaml.org',
        Ruby: 'ruby-lang.org',
        Python: 'python.org',
        Perl: 'use.perl.org' }
}
```

## 配置纯量

纯量是最基本的、不可再分的值。

### 布尔值

布尔值用true和false表示。

```yml
isSet: true
```

### 整数 & 浮点数

数值直接以字面量的形式表示

- 整数

  ```yml
  number: 12
  ```

- 浮点数

  ```yml
  number: 12.30
  ```

### NULL

NULL比较特殊，NULL使用 `~` 来表示

```yml
# 注意区分 ~ 和 `
isSet: ~
```

### 时间

时间采用 ISO8601 格式。

```yml
iso8601: 2001-12-14t21:59:43.10-05:00
```

JavaScript

```javascript
{ iso8601: new Date('2001-12-14t21:59:43.10-05:00') }
```

### 日期

日期采用复合 iso8601 格式的年、月、日表示。

```yml
date: 1976-07-31
```

JavaScript

```javascript
{ date: new Date('1976-07-31') }
```

### 字符串

字符串默认不使用引号表示。

```yml
str: 这是一行字符串
```

如果字符串之中包含空格或特殊字符，需要放在引号之中。

```yml
str: '内容： 字符串'
```

单引号和双引号都可以使用，双引号不会对特殊字符转义。

```yml
s1: '内容\n字符串'
s2: "内容\n字符串"
```

JavaScript

```javascript
{
    s1: '内容\\n字符串',
    s2: '内容\n字符串'
}
```

单引号之中如果还有单引号，必须连续使用两个单引号转义。

```yml
str: 'labor''s day'
```

JavaScript

```javascript
{ str: 'labor\'s day' }
```

字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格。

```yml
str: 这是一段
 多行
 字符串
```

JavaScript

```javascript
{ str: '这是一段 多行 字符串' }
```

多行字符串可以使用|保留换行符，也可以使用>折叠换行。

```yml
this: |
  Foo
  Bar
that: >
  Foo
  Bar
```

JavaScript

```javascript
{ this: 'Foo\nBar\n', that: 'Foo Bar\n' }
```

`+` 表示保留文字块末尾的换行，`-` 表示删除字符串末尾的换行。

```yml
s1: |
  Foo
s2: |+
  Foo
s3: |-
  Foo
```

JavaScript

```javascript
{ s1: 'Foo\n', s2: 'Foo\n\n\n', s3: 'Foo' }
```

字符串之中可以插入 HTML 标记。

```yml
message: |
  <p style="color: red">
    段落
  </p>
```

JavaScript

```javascript
{ message: '\n<p style="color: red">\n  段落\n</p>\n' }
```

### 类型转换

YAML 允许使用两个感叹号，强制转换数据类型。

```yml
e: !!str 123
f: !!str true
```

JavaScript

```javascript
{ e: '123', f: 'true' }
```

## 引用

锚点 `&`和别名 `*`，可以用来引用。

```yml
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
```

等同于：

```yml
defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```

`&` 用来建立锚点（defaults），`<<` 表示合并到当前数据，`*` 用来引用锚点。

其他例子：

```yml
- &anchor Steve
- Clark
- Brian
- Oren
- *anchor
```

JavaScript

```js
[ 'Steve', 'Clark', 'Brian', 'Oren', 'Steve' ]
```
