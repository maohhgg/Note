## 2. 代码规范

### 2.1 Java语言规范

#### 2.1.1  绝对不忽略异常

以正确的方式处理异常。例如：
```java
public void setUserId(String id) {
	try {
    	mUserId = Integer.parseInt(id);
	} catch (NumberFormatException e) { }
}
```
这里将会给开发者和用户带来不清楚的消息。如果发生错误并捕获到异常时，这种写法会使得开发者很难debug并且会给用户带来困惑，如有必要提示用户并且在console记录信息让我们去debug。应如下：

```java
public void setCount(String count) {
	try {
    	count = Integer.parseInt(id);
	} catch (NumberFormatException e) {
		count = 0;
    	Log.e(TAG, "There was an error parsing the count " + e);
    	DialogFactory.showErrorMessage(R.string.error_message_parsing_count);
	}
}
```
适当处理如下的错误：

* 显示一个消息给用户提示他们出现了一个错误。
* 设置时有默认的选项
* 抛出一个合适的异常



#### 2.1.2 不要捕捉普通异常

捕捉异常时不应该有下面的情况：

```java
public void openCustomTab(Context context, Uri uri) {
	Intent intent = buildIntent(context, uri);
	try {
    	context.startActivity(intent);
	} catch (Exception e) {
    	Log.e(TAG, "There was an error opening the custom tab " + e);
	}
}
```

为什么?

*通常不要这样做，捕捉或者抛出（最好不要抛出，因为它有可能包是错误）普通异常是不恰当的，这样是非常危险的，你有可能捕捉到系统级别的错误（像RuntimeExceptions和ClassCastException），而你的代码就覆盖了他们的提示。意味着有时有新类型的异常发生在这段代码，编译器不会提示你去处理新类型的异常，多数情况下不应该用同样的方式处理不同的异常。* - 遵从Android代码规范

#### 2.1.3 异常分组

在相同位置有不同的异常发生，为了增加代码可读性和避免重复代码，你应该如下这样做：

```java
public void openCustomTab(Context context, @Nullable Uri uri) {
	Intent intent = buildIntent(context, uri);
	try {
    	context.startActivity(intent);
	} catch (ActivityNotFoundException e) {
    	Log.e(TAG, "There was an error opening the custom tab " + e);
	} catch (NullPointerException e) {
    	Log.e(TAG, "There was an error opening the custom tab " + e);
	} catch (SomeOtherException e) {
		// Show some dialog
    }
}
```

也可以这样做:
```java
public void openCustomTab(Context context, @Nullable Uri uri) {
	Intent intent = buildIntent(context, uri);
	try {
    	context.startActivity(intent);
	} catch (ActivityNotFoundException e | NullPointerException e) {
    	Log.e(TAG, "There was an error opening the custom tab " + e);
	} catch (SomeOtherException e) {
		// Show some dialog
    }
}
```
#### 2.1.4 使用try-catch来抛出异常

使用try-catch语句改善发生异常代码的可读性。当异常出现是，更容易调试和处理错误。



#### 2.1.6 禁止全部导入

当导入库时，不应该导入全部包，比如：

```java
import android.support.v7.widget.*;
```

相反，应该这样：

```java
import android.support.v7.widget.RecyclerView;
```

#### 2.1.7 不要保留没使用的库

有时删除一些代码而有的库不再使用，那么应该删除导入这些库的代码。


### 2.2 Java代码规范

#### 2.2.1 字段定义和命名


所有属性应在该文件的顶部被声明, 遵循下面的规则:

* 不是静态私有属性不要使用`m`开始，正确示例：

```
    userSignedIn, userNameText, acceptButton
```

错误示范:

```
mUserSignedIn, mUserNameText, mAcceptButton
```

* 静态私有属性使用需要使用`s`开始,正确示例：

```
someStaticField, userNameText
```

错误示范:
```
sSomeStaticField, sUserNameText
```

* 其他属性使用小写字母开头：

```java
int numOfChildren;
String username;
```

* 静态的终态（final）属性（即常量），所有字母要大写。

```java
private static final int PAGE_COUNT = 0;
```

属性名称不应定义模糊，比如：

```java
int e; //列表中的元素的数目
```

使用他的作用作为名称，而不是留下注释来说明。这样更好

```java
int numberOfElements;
```

#### 2.2.1.2 View属性名

当属性涉及views时，名称的最后一个单词应该是`view`,比如：

| View           | Name              |
|----------------|-------------------|
| TextView       | usernameView      |
| Button         | acceptLoginView   |
| ImageView      | profileAvatarView |
| RelativeLayout | profileLayout     |

这样我们可以轻松地确认属性属于谁。例如，有一个字段中命名为**用户**，他是一个非常模糊的名称。给它的命名为usernameView，userAvatarView或userProfieLayout有助于弄清楚属性属于谁。

在以前通常使用view的类型作为结尾（acceptLoginButton），但当views发生改变时，通常会忘记回到java类中去修改属性名称。

#### 2.2.2 避免与容器类型相同

当创建一个集合变量时，我们命名应该避免使用容器类型。比如，我们有一个包含用户列表ArrayList

应该这样命名：
```java
List<String> userIds = new ArrayList<>();
```

而不应该：

```java
List<String> userIdList = new ArrayList<>();
```

当我们更改容器时，我们通常会忘记更改名称，就像view的命名，他是不必要的。正确的名称包含他的属性信息就足够了。


#### 2.2.3 避免相似的名称

命名相似名称的变量、方法或类，会使其他开发人员阅读代码时产生混淆，比如：

```
hasUserSelectedSingleProfilePreviously

hasUserSelectedSignedProfilePreviously
```
乍看之下可能很难找到他们之间的区别。一个更清晰的命名方式可以使开发人员更容易操作你的代码。

#### 2.2.4 连续数字命名

当Android Studio给我们自动创建代码时，他会使用连续数字命名，这是参数命名是可怕的！比如，这样不太好：
```java
public void doSomething(String s1, String s2, String s3)
```

没有阅读注释很难了解这些参数，相反：
```java
public void doSomething(String userName, String userEmail, String userId)
```

这样就很容易理解，现在我们根据代码就能够清晰的了解参数的意义🙂

#### 2.2.5 易识别的命名

当命名属性、方法和类时应该：

* 容易阅读：有效的命名能够阅读同时就能理解他表达的意思，减少去理解他的意义消耗的时间。

* 容易发音：容易朗读的命名，可以避免当你和他人对话时你试图发音一个命名的尴尬。

* 容易寻找：没有什么比试图在一个类中寻找一个被拼写错了（或严重糟糕）的方法或变量更糟糕了。如果我们试图找到**搜索用户**的方法，那么当搜索`search`时他就应该出现。

* 不要使用匈牙利表示法：匈牙利表示法违背上述提出的三点，所以绝对不能用！

#### 2.2.6 缩写的单词必须大写

所有的包含缩写的类、变量名等都应该使用大写来处理。比如：

| Good            | bad             |
|-----------------|-----------------|
| setUserId       | setUserID       |
| String uri      | String URI      |
| int id          | int ID          |
| parseHtml       | parseHTML       |
| generateXmlFile | generateXMLFile |

#### 2.2.7 避免特殊整理变量申明

所有的变量申明不应该使用特殊方式来对齐。比如：

正常的代码:
```java
private int userId = 8;
private int count = 0;
private String username = "hitherejoe";
```
避免这样:
```java
private String username = "hitherejoe";
private int userId      = 8;
private int count       = 0;
```
这种声明方式在阅读时可能难以理解空白的字符串。

#### 2.2.8 缩进

对于代码块，缩进应该使用4个空格：
```java
if (userSignedIn) {
    count = 1;
}
```
而当包裹多行，缩进应该使用8个空格：

```java
    String userAboutText =
            "This is some text about the user and it is pretty long, can you see!"
```
### 2.2.9 If-Statements

#### 2.2.9.1 大括号规范

大括号通常应该和他之前的代码同行，避免这样：

```java
class SomeClass
{
	private void someFunction()
	{
    	if (isSomething)
    	{

    	}
    	else if (!isSomethingElse)
    	{

    	}
    	else
    	{

    	}
	}
}
```

这样更好:

```java
class SomeClass {
	private void someFunction() {
    	if (isSomething) {

    	} else if (!isSomethingElse) {

    	} else {

    	}
	}
}
```

特别增加一行完全没有必要，也不会更容易阅读代码。

#### 2.2.9.2 单行if语句

有时使用单行if语句是可以的，比如：

```java
if (user == null) return false;
```

然而，这只能是在简单的操作时，而这样的代码会更适合用大括号：

```java
if (user == null) throw new IllegalArgumentExeption("Oops, user object is required.");
```
这样更好：
```java
if (user == null) {
	throw new IllegalArgumentExeption("Oops, user object is required.");
}
```
#### 2.2.9.3 条件嵌套

可能的情况下，如果条件能合并，就应该避免复杂的嵌套。比如：

Do:

```java
if (userSignedIn && userId != null) {

}
```

Try to avoid:

```java
if (userSignedIn) {
    if (userId != null) {

    }
}
```

这使得语句更容易阅读并且嵌套除去了额外的行。
