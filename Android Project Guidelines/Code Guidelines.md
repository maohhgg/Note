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

#### 2.2.9.4 三元运算

在合适的地方，使用三元运算简化操作，比如，这样更容易阅读：

```java
userStatusImage = signedIn ? R.drawable.ic_tick : R.drawable.ic_cross;
```
而且使用更少的行比下面这样：

```java
if (signedIn) {
    userStatusImage = R.drawable.ic_tick;
} else {
    userStatusImage = R.drawable.ic_cross;
}
```

### 2.2.10 注释

#### 2.2.10.1 注释实践

Android代码风格：

**@Override:** @Override 注释必须是方法声明覆盖了实现的接口或父类的方法。举个例子，如果你使用@inheritdocs Javadoc标签，从一个类派生（不是实现），你也必须标注该方法@Overrides了父类的方法

**@SuppressWarnings:** @SuppressWarnings 注释应该只在它是不出现警告的情况下使用。如果警告你确定它不是问题，就应该使用@SuppressWarning。所以确保所有警告的代码没有问题。


----------

注释尽可能多。比如，当你一个属性可能是空时可以用@Nullable注释，例子：
```java
@Nullable TextView userNameText;

private void getName(@Nullable String name) { }
```

#### 2.2.10.2 注释风格

所应用方法或类的注释应该在声明中定义，每个注释单独一行：

```java
@Annotation
@AnotherAnnotation
public class SomeClass {

  @SomeAnotation
  public String getMeAString() {

  }

}
```

当给属性添加注释时，应该确保注释在同一行上且有多余空间。比如：

```java
@Bind(R.id.layout_coordinator) CoordinatorLayout coordinatorLayout;

@Inject MainPresenter mainPresenter;
```
我们这样做是因为它使语句更易于阅读。例如，`@Inject SomeComponent mSomeName`意味着这个变量代表此组件

#### 2.2.11 限制变量的作用域

局部变量的范围应保持在最低限度（Effective Java Item 29）。通过这样做，增加代码的可读性和可维护性，减少出错的可能性。每个变量都应该在封闭在所用的代码块中，外部不可以访问。

局部变量应当在其第一次使用时声明，每个局部变量申明时都要初始化。如果你没有足够的信息来初始化变量，你应该推迟申明变量，直到你要使用他。-- Android代码风格


#### 2.2.12 未使用的元素

所有未使用的**属性** 、**导入（imports）** 、**方法** 和**类**应该从代码中删除，除非后面注释有它有任何具体的理由。

#### 2.2.13 Order Import Statements 导入顺序

使用Android Studio时，导入会自动排序。而其他情况时也许不会自动排序，这时我们应该手动排序根据：

1. Android库的导入
2. 第三方库的导入
3. J2EE和J2SE的导入
4. 当前项目文件中的导入

**提示：**

* 导入应该按照字母顺序排列，大写字母应在小写之前（比如Z在a前）。

* 每种分组之间应该用空行隔开，(android, com, JUnit, net, org, java, javax)


#### 2.2.14 Logging

Logging should be used to log useful error messages and/or other information that may be useful during development.


| Log                               | Reason      |
|-----------------------------------|-------------|
| Log.v(String tag, String message) | verbose     |
| Log.d(String tag, String message) | debug       |
| Log.i(String tag, String message) | information |
| Log.w(String tag, String message) | warning     |
| Log.e(String tag, String message) | error       |


We can set the `Tag` for the log as a `static final` field at the top of the class, for example:


    private static final String TAG = MyActivity.class.getName();

All verbose and debug logs must be disabled on release builds. On the other hand - information, warning and error logs should only be kept enabled if deemed necessary.


    if (BuildConfig.DEBUG) {
        Log.d(TAG, "Here's a log message");
    }

**Note:** Timber is the preferred logging method to be used. It handles the tagging for us, which saves us keeping a reference to a TAG.

#### 2.2.15 Field Ordering

Any fields declared at the top of a class file should be ordered in the following order:

1. Enums
2. Constants
3. Dagger Injected fields
4. Butterknife View Bindings
5. private global variables
6. public global variables

For example:

	public static enum {
		ENUM_ONE, ENUM_TWO
	}

	public static final String KEY_NAME = "KEY_NAME";
	public static final int COUNT_USER = 0;

	@Inject SomeAdapter someAdapter;

	@BindView(R.id.text_name) TextView nameText;
	@BindView(R.id.image_photo) ImageView photoImage;

	private int userCount;
	private String errorMessage;

	public int someCount;
	public String someString;

Using this ordering convention helps to keep field declarations grouped, which increases both the locating of and readability of said fields.

#### 2.2.16 Class member ordering


To improve code readability, it’s important to organise class members in a logical manner. The following order should be used to achieve this:


1. Constants
2. Fields
3. Constructors
4. Override methods and callbacks (public or private)
5. Public methods
6. Private methods
7. Inner classes or interfaces

For example:


    public class MainActivity extends Activity {

        private int count;

        public static newInstance() { }

        @Override
        public void onCreate() { }

        public setUsername() { }

        private void setupUsername() { }

        static class AnInnerClass { }

        interface SomeInterface { }

    }

Any lifecycle methods used in Android framework classes should be ordered in the corresponding lifecycle order. For example:


    public class MainActivity extends Activity {

        // Field and constructors

        @Override
        public void onCreate() { }

        @Override
        public void onStart() { }

        @Override
        public void onResume() { }

        @Override
        public void onPause() { }

        @Override
        public void onStop() { }

        @Override
        public void onRestart() { }

        @Override
        public void onDestroy() { }

        // public methods, private methods, inner classes and interfaces

    }

#### 2.2.17 Method parameter ordering

When defining methods, parameters should be ordered to the following convention:

    public Post loadPost(Context context, int postId);


    public void loadPost(Context context, int postId, Callback callback);

**Context** parameters always go first and **Callback** parameters always go last.

#### 2.2.18 String constants, naming, and values

When using string constants, they should be declared as final static and use the follow conventions:

[Strings table]

#### 2.2.19 Enums

Enums should only be used where actually required. If another method is possible, then that should be the preferred way of approaching the implementation. For example:

Instead of this:


    public enum SomeEnum {
        ONE, TWO, THREE
    }

Do this:

    private static final int VALUE_ONE = 1;
    private static final int VALUE_TWO = 2;
    private static final int VALUE_THREE = 3;

#### 2.2.20 Arguments in fragments and activities

When we pass data using an Intent or Bundle, the keys for the values must use the conventions defined below:

**Activity**

Passing data to an activity must be done using a reference to a KEY, as defined as below:


    private static final String KEY_NAME = "com.your.package.name.to.activity.KEY_NAME";

**Fragment**

Passing data to a fragment must be done using a reference to an EXTRA, as defined as below:


    private static final String EXTRA_NAME = "EXTRA_NAME";

When creating new instances of a fragment or activity that involves passing data, we should provide a static method to retrieve the new instance, passing the data as method parameters. For example:

**Activity**

    public static Intent getStartIntent(Context context, Post post) {
        Intent intent = new Intent(context, CurrentActivity.class);
        intent.putParcelableExtra(EXTRA_POST, post);
        return intent;
    }

**Fragment**

    public static PostFragment newInstance(Post post) {
        PostFragment fragment = new PostFragment();
        Bundle args = new Bundle();
        args.putParcelable(ARGUMENT_POST, post);
        fragment.setArguments(args)
        return fragment;
    }

#### 2.2.21 Line Length Limit

Code lines should exceed no longer than 100 characters, this makes the code more readable. Sometimes to achieve this, we may need to:


- Extract data to a local variable
- Extract logic to an external method
- Line-wrap code to separate a single line of code to multiple lines

**Note:** For code comments and import statements it’s ok to exceed the 100 character limit.

#### 2.2.21.1 Line-wrapping techniques

When it comes to line-wraps, there’s a few situations where we should be consistent in the way we format code.

**Breaking at Operators**

When we need to break a line at an operator, we break the line before the operator:


    int count = countOne + countTwo - countThree + countFour * countFive - countSix
            + countOnANewLineBecauseItsTooLong;

If desirable, you can always break after the `=` sign:


    int count =
            countOne + countTwo - countThree + countFour * countFive + countSix;

**Method Chaining**

When it comes to method chaining, each method call should be on a new line.

Don’t do this:


    Picasso.with(context).load("someUrl").into(imageView);

Instead, do this:


    Picasso.with(context)
            .load("someUrl")
            .into(imageView);

**Long Parameters**

In the case that a method contains long parameters, we should line break where appropriate. For example when declaring a method we should break after the last comma of the parameter that fits:


    private void someMethod(Context context, String someLongStringName, String text,
                                long thisIsALong, String anotherString) {               
    }             

And when calling that method we should break after the comma of each parameter:


    someMethod(context,
            "thisIsSomeLongTextItsQuiteLongIsntIt",
            "someText",
            01223892365463456,
            "thisIsSomeLongTextItsQuiteLongIsntIt");


#### 2.2.22 Method spacing

There only needs to be a single line space between methods in a class, for example:

Do this:


    public String getUserName() {
        // Code
    }

    public void setUserName(String name) {
        // Code
    }

    public boolean isUserSignedIn() {
        // Code
    }

Not this:


    public String getUserName() {
        // Code
    }


    public void setUserName(String name) {
        // Code
    }


    public boolean isUserSignedIn() {
        // Code
    }

### 2.2.23 Comments

#### 2.2.23.1 Inline comments

Where necessary, inline comments should be used to provide a meaningful description to the reader on what a specific piece of code does. They should only be used in situations where the code may be complex to understand. In most cases however, code should be written in a way that it easy to understand without comments 🙂

**Note:** Code comments do not have to, but should try to, stick to the 100 character rule.

#### 2.2.23.2 JavaDoc Style Comments


Whilst a method name should usually be enough to communicate a methods functionality, it can sometimes help to provide JavaDoc style comments. This helps the reader to easily understand the methods functionality, as well as the purpose of any parameters that are being passed into the method.


    /**
     * Authenticates the user against the API given a User id.
     * If successful, this returns a success result
     *
     * @param userId The user id of the user that is to be authenticated.
     */

#### 2.2.23.3 Class comments

When creating class comments they should be meaningful and descriptive, using links where necessary. For example:


    /**
      * RecyclerView adapter to display a list of {@link Post}.
      * Currently used with {@link PostRecycler} to show the list of Post items.
      */

Don’t leave author comments, these aren’t useful and provide no real meaningful information when multiple people are to be working on the class.


    /**
      * Created By Joe 18/06/2016
      */

### 2.2.24 Sectioning code

#### 2.2.24.1 Java code

If creating ‘sections’ for code, this should be done using the following approach, like this:


    public void method() { }

    public void someOtherMethod() { }

    /********* Mvp Method Implementations  ********/

    public void anotherMethod() { }

    /********* Helper Methods  ********/

    public void someMethod() { }

Not like this:


    public void method() { }

    public void someOtherMethod() { }

    // Mvp Method Implementations

    public void anotherMethod() { }

This makes sectioned methods easier to located in a class.

#### 2.2.24.2 Strings file

String resources defined within the string.xml file should be section by feature, for example:


    // User Profile Activity
    <string name="button_save">Save</string>
    <string name="button_cancel">Cancel</string>

    // Settings Activity
    <string name="message_instructions">...</string>

Not only does this help keep the strings file tidy, but it makes it easier to find strings when they need altering.

#### 2.2.24.3 RxJava chaining

When chaining Rx operations, every operator should be on a new line, breaking the line before the period `.` . For example:


    return dataManager.getPost()
                .concatMap(new Func1<Post, Observable<? extends Post>>() {
                    @Override
                     public Observable<? extends Post> call(Post post) {
                         return mRetrofitService.getPost(post.id);
                     }
                })
                .retry(new Func2<Integer, Throwable, Boolean>() {
                     @Override
                     public Boolean call(Integer numRetries, Throwable throwable) {
                         return throwable instanceof RetrofitError;
                     }
                });

This makes it easier to understand the flow of operation within an Rx chain of calls.

### 2.2.25 Butterknife

#### 2.2.25.1 Event listeners

Where possible, make use of Butterknife listener bindings. For example, when listening for a click event instead of doing this:


    mSubmitButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View v) {
            // Some code here...
        }
      };

Do this:


    @OnClick(R.id.button_submit)
    public void onSubmitButtonClick() { }


## 2.3 XML Style Rules

### 2.3.1 Use self=-closing tags

When a View in an XML layout does not have any child views, self-closing tags should be used.

Do:


    <ImageView
        android:id="@+id/image_user"
        android:layout_width="90dp"
        android:layout_height="90dp" />

Don’t:


    <ImageView
        android:id="@+id/image_user"
        android:layout_width="90dp"
        android:layout_height="90dp">
    </ImageView>


### 2.3.2 Resource naming

All resource names and IDs should be written using lowercase and underscores, for example:


    text_username, activity_main, fragment_user, error_message_network_connection

The main reason for this is consistency, it also makes it easier to search for views within layout files when it comes to altering the contents of the file.

#### 2.3.2.1 ID naming

All IDs should be prefixed using the name of the element that they have been declared for.

| Element        | Prefix    |
|----------------|-----------|
| ImageView      | image_    |
| Fragment       | fragment_ |
| RelativeLayout | layout_   |
| Button         | button_   |
| TextView       | text_     |
| View           | view_     |

For example:


    <TextView
        android:id="@+id/text_username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />


Views that typically are only one per layout, such as a toolbar, can simply be given the id of it's view type. E.g.```toolbar```.

#### 2.3.2.2 Strings

All string names should begin with a prefix for the part of the application that they are being referenced from. For example:

| Screen                | String         | Resource Name             |
|-----------------------|----------------|---------------------------|
| Registration Fragment | “Register now” | registration_register_now |
| Sign Up Activity      | “Cancel”       | sign_up_cancel            |
| Rate App Dialog       | “No thanks”    | rate_app_no_thanks        |

If it’s not possible to name the referenced like the above, we can use the following rules:

| Prefix  | Description                                  |
|---------|----------------------------------------------|
| error_  | Used for error messages                      |
| title_  | Used for dialog titles                       |
| action_ | Used for option menu actions                 |
| msg_    | Used for generic message such as in a dialog |
| label_  | Used for activity labels                     |

Two important things to note for String resources:

 - String resources should never be reused across screens. This can cause issues when it comes to changing a string for a specific screen. It saves future complications by having a single string for each screens usage.

 - String resources should **always** be defined in the strings file and never hardcoded in layout or class files.

#### 2.3.2.3 Styles and themes

When defining both Styles & Themes, they should be named using UpperCamelCase. For example:


    AppTheme.DarkBackground.NoActionBar
    AppTheme.LightBackground.TransparentStatusBar

    ProfileButtonStyle
    TitleTextStyle


### 2.3.3 Attributes ordering

Ordering attributes not only looks tidy but it helps to make it quicker when looking for attributes within layout files. As a general rule,


1. View Id
2. Style
3. Layout width and layout height
4. Other `layout_` attributes, sorted alphabetically
5. Remaining attributes, sorted alphabetically

For example:

    <Button
        android:id="@id/button_accept"
        style="@style/ButtonStyle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentStart="true"
        android:padding="16dp"
        android:text="@string/button_skip_sign_in"
        android:textColor="@color/bluish_gray" />

Note: This formatting can be carried out by using the format feature in android studio -

`cmd + shift + L`

Doing this makes it easy to navigate through XML attributes when it comes to making changes to layout files.


## 2.4 Tests style rules

### 2.4.1 Unit tests

Any Unit Test classes should be written to match the name of the class that the test are targeting, followed by the Test suffix. For example:

| Class                | Test Class               |
|----------------------|--------------------------|
| DataManager          | DataManagerTest          |
| UserProfilePresenter | UserProfilePresenterTest |
| PreferencesHelper    | PreferencesHelperTest    |

All Test methods should be annotated with the `@Test` annotation, the methods should be named using the following template:


    @Test
    public void methodNamePreconditionExpectedResult() { }

So for example, if we want to check that the signUp() method with an invalid email address fails, the test would look like:


    @Test
    public void signUpWithInvalidEmailFails() { }

Tests should focus on testing only what the method name entitles, if there’s extra conditions being tested in your Test method then this should be moved to it’s own individual test.

If a class we are testing contains many different methods, then the tests should be split across multiple test classes - this helps to keep the tests more maintainable and easier to locate. For example, a DatabaseHelper class may need to be split into multiple test classes such as :


    DatabaseHelperUserTest
    DatabaseHelperPostsTest
    DatabaseHelperDraftsTest

### 2.4.2 Espresso tests

Each Espresso test class generally targets an Activity, so the name given to it should match that of the targeted Activity, again followed by Test. For example:

| Class                | Test Class               |
|----------------------|--------------------------|
| MainActivity         | MainActivityTest         |
| ProfileActivity      | ProfileActivityTest      |
| DraftsActivity       | DraftsActivityTest       |

When using the Espresso API, methods should be chained on new lines to make the statements more readable, for example:


    onView(withId(R.id.text_title))
            .perform(scrollTo())
            .check(matches(isDisplayed()))

Chaining calls in this style not only helps us stick to less than 100 characters per line but it also makes it easy to read the chain of events taking place in espresso tests.
