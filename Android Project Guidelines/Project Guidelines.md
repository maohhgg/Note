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
