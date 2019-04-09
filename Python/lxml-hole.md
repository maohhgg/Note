---
title: lxml xpath使用中的一些坑
date: 2017/12/15
tag:
  - python
  - lxml
  - XPath
urlname: xpath-with-lxml
---

#### 根据 class 查找

如果Dom含有多个ClassName，当使用`@class`用一个ClassName查找Dom时，是没有结果的。必须使用包含全部的ClassName。

```python
# 例 <div class="demo test"></div>
etree.xpath('*[@class="demo"]') # None
etree.xpath('*[@class="demo test"]') # <class 'lxml.etree._Element'>
```
<!--more-->
要使用单个className查找，应使用下面的方法 

```python
# 例 <div class="demo test"></div>
etree.xpath('*[contains(@class,"demo")]') # <class 'lxml.etree._Element'>
# 完全匹配也可
etree.xpath('*[contains(@class,"demo test")]') # <class 'lxml.etree._Element'>
```

#### ElementUnicodeResult类的getparent()

html数据
```html
<p class="para">
    如果可选的第三个参数
    <code class="parameter">strict</code>
    为
    <strong>
        <code>TRUE</code>
    </strong>
    ，则
    <span class="function">
        <strong>array_search()</strong>
    </span> 
    将在
    <code class="parameter">haystack</code>
    中检查
    <em class="emphasis">完全相同</em>的元素。这意味着同样严格比较
    <code class="parameter">haystack</code> 里
    <code class="parameter">needle</code> 的
    <a href="language.types.php" class="link">类型</a>
    ，并且对象需是同一个实例。
</p>
```
方法
```python
def get_code(self, node, active=False):
    c = ''
    if isinstance(node, etree._ElementUnicodeResult):
        c = str(node).replace('\n', '').replace(' ', '').strip()
        p = node.getparent()
        print(p.tag + ' : ' + str(p.attrib) +'  | content: '+ c)
        return c
    if isinstance(node, etree._Element):
        for item in node.xpath('node()'):
            c += str(get_code(item, active))
        return c
```
使用上面的方法递归读取每个节点的文本内容，xpath('node()')匹配文本为[lxml.etree.ElementUnicodeResult](http://lxml.de/api/lxml.etree._ElementUnicodeResult-class.html)对象，节点为[lxml.etree.Element](http://lxml.de/api/lxml.etree._Element-class.html)对象。

混合排版的文本ElementUnicodeResult对象getparent()不能正确获取父节点，前面有兄弟节点时getparent()获取到的时前边的兄弟节点。比如文本`为` `，则` `将在` `中检查` 都是获取到文字前边的的节点

前边方法的输出
```
p : {'class': 'para'}  content: 如果可选的第三个参数
code : {'class': 'parameter'}  content: strict
code : {'class': 'parameter'}  content: 为
code : {}  content: TRUE
strong : {}  content: ，则
strong : {}  content: array_search()
span : {'class': 'function'}  content: 将在
code : {'class': 'parameter'}  content: haystack
code : {'class': 'parameter'}  content: 中检查
em : {'class': 'emphasis'}  content: 完全相同
em : {'class': 'emphasis'}  content: 的元素。这意味着同样严格比较
code : {'class': 'parameter'}  content: haystack
code : {'class': 'parameter'}  content: 里
code : {'class': 'parameter'}  content: needle
code : {'class': 'parameter'}  content: 的
a : {'href': 'http://php.net/manual/zh/language.types.php', 'class': 'link'}  content: 类型
a : {'href': 'http://php.net/manual/zh/language.types.php', 'class': 'link'}  content: ，并且对象需是同一个实例。

```

我的目的是根据他的父节点的标签 判断文字是**粗体** 还是*斜体*
期待结果应该为
```md
如果可选的第三个参数`strict` 为 **`TRUE`**，则**array_searck**中检查*完全相同*的元素。这意味着同样严格比较`haystack`里`needle`的类型，并且对象需是同一个实例。
```
```php
<?php 
$array = array( 0 => 'blue' , 1 => 'red' , 2 => 'green' , 3 => 'red' ); 

$key = array_search ( 'green' , $array ); // $key = 2; 
$key = array_search ( 'red' , $array ); // $key = 1; 
?> 
```
搜索一圈没找到解决。临时解决方法传入父节点，递归每层深度的父节点是一样的


问题发布在[segmentfault](https://segmentfault.com/q/1010000012449373)，等待其他大牛来解答。实际[代码](https://github.com/maohhgg/WebCrawler/blob/master/Item/php_net.py#L91)

