# Android 项目指南
---------------------------

## 1. 项目指南

### 1.1 项目结构

当启动项目时，项目应保持如下的结构：

	src/androidTest
	src/test
	src/commonTest
	src/main


**androidTest** - 功能测试文件夹  
**test** - 单元测试文件夹    
**commonTest** - 文件夹包含AndroidTest和Test的共享代码  
**main** - 项目代码

当修改或增加新的功能应该保持如上项目的结构。

采用这样的结构能是我们的项目代码和测试相关的代码分离开。CommonTest目录允许我们分享功能测试和单元测试的类，例如模型创建和Dagger测试配置类。

### 1.2 文件命名

#### 1.2.1 类文件

任何类都应该使用驼峰命名法来命名，例如：

	AndroidActivity, NetworkHelper, UserFragment, PerActivity

任何继承Android框架组件的类都**必须**使用组件名结尾，例如：

    UserFragment, SignUpActivity, RateAppDialog, PushNotificationServer, NumberView

使用驼峰命名法命名可以帮助我们分割单词，使得方便阅读。使用框架组件结尾命名的类可以清楚的知道这个类的用。例如，如果你想修改RegistrationDialog文件，使用命名方式命名的可以轻松找到他。

#### 1.2.1 资源文件

命名资源文件时应该全部使用小写并用`_`分割单词而不是使用空格，例如：

	activity_main, fragment_user, item_post

这样的习惯通常便于在布局文件中找到你想要的，在Android Studio中，布局文件按activity字母顺序排序,fragment和其他布局成一组，这样方便寻找一个文件。除此之外，文件名以组件名开头可以知道是哪个组件或类使用了他们。


#### 1.2.2.1 Drawable Files

Drawable资源文件通常使用**ic_**作为前缀连接资源的大小和颜色，例如，24dp的白色确认图标

	ic_accept_24dp_white

那么48dp的黑色取消图标：

	ic_cancel_48dp_black

这样的命名习惯我们根据drawable文件名字可以知道他的内容，如果文件名中没有大小和颜色，开发者必须打开Drawable文件才能知道。文件名节约了一点点时间:)

其他drawable文件应该使用相应的前缀，例如：

| Type       | Prefix    | Example                |
|------------|-----------|------------------------|
| Selector   | selector_ | selector_button_cancel |
| Background | bg_       | bg_rounded_button      |
| Circle     | circle_   | circle_white           |
| Progress   | progress_ | progress_circle_purple |
| Divider    | divider_  | divider_grey           |

在Android Studio中这一规范帮助不同类的分组，方便了解哪一类使用了资源。例如，命名为button_cancel的资源，他可能是selector资源也有可能是button资源。而当前命名规范可以清除可能出现的模糊定义。

当创建不同状态selector资源时，他们的文件名应该有相应后缀。

| State    | Suffix    | Example             |
|----------|-----------|---------------------|
| Normal   | _normal   | btn_accept_normal   |
| Pressed  | _pressed  | btn_accept_pressed  |
| Focused  | _focused  | btn_accept_focused  |
| Disabled | _disabled | btn_accept_disabled |
| Selected | _selected | btn_accept_selected |

#### 1.2.2.2 Layout Files

当命名Layout文件，文件名开始应该为创建他们的Android组件的名称。例如：

| Component        | Class Name      | Layout Name       |
|------------------|-----------------|-------------------|
| Activity         | MainActivity    | activity_main     |
| Fragment         | MainFragment    | fragment_main     |
| Dialog           | RateDialog      | dialog_rate       |
| Widget           | UserProfileView | view_user_profile |
| AdapterView Item | N/A             | item_follower     |

这方法不仅使得很容易找到目录下不同层次的文件，而且了解布局文件属于哪个类时也有帮助。

**Note:** 如果使用合并标签创建布局则应该使用layout_前缀。

#### 1.2.2.3 Menu Files

菜单文件不需要menu_作为文件名前缀。因为他们已经在menu包文件目录下了，不需要额外的前缀。

#### 1.2.2.4 Values Files

所有的资源文件应该使用负数形式，例如：

	attrs.xml, strings.xml, styles.xml, colors.xml, dimens.xml


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



#### 2.1.2 Never catch generic exceptions

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
