---
title: Intent （Android笔记二）
date: 2016/9/9
tag:
  - Android
urlname: android-note-the-intent
---

# Android笔记

--------------------------------------------------------------------------------

## Intent 和 Intent 过滤器

### Intent

Intent 是一个消息传递对象，您可以使用它从其他应用组件请求操作。尽管 Intent 可以通过多种方式促进组件之间的通信，但其基本用例主要包括以下三个

- 启动 Activity：

  1. `startActivity()`
  2. `startActivityForResult()`

- 启动服务：

  1. `startService()`
  2. `bindService()`

- 传递广播：

  1. `sendBroadcast()`
  2. `sendOrderedBroadcast()`
  3. `sendStickyBroadcast()`

### Intent 类型

- 显式 Intent

  按名称（完全限定类名）指定要启动的组件。通常，您会在自己的应用中使用显式 Intent 来启动组件，这是因为您知道要启动的 Activity 或服务的类名。例如，启动新 Activity 以响应用户操作，或者启动服务以在后台下载文件。

  ```java
  Intent downloadIntent = new Intent(this, DownloadService.class);
  downloadIntent.setData(Uri.parse(fileUrl));
  startService(downloadIntent);
  ```

- 隐式 Intent

  不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。例如，如需在地图上向用户显示位置，则可以使用隐式 Intent，请求另一具有此功能的应用在地图上显示指定的位置。

  ```java
  Intent sendIntent = new Intent();
    sendIntent.setAction(Intent.ACTION_SEND);
    sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
    sendIntent.setType("text/plain");

    if (sendIntent.resolveActivity(getPackageManager()) != null) {
        startActivity(sendIntent);
    }
  ```

创建显式 Intent 启动 Activity 或服务时，系统将立即启动 Intent 对象中指定的应用组件。

创建隐式 Intent 时，Android 系统通过将 Intent 的内容与在设备上其他应用的清单文件中声明的 Intent 过滤器进行比较，从而找到要启动的相应组件。Intent如果 Intent 与 Intent 过滤器匹配，则系统将启动该组件，并将其传递给对象。如果多个 Intent 过滤器兼容，则系统会显示一个对话框，支持用户选取要使用的应用。

> 警告：为了确保应用的安全性，启动 Service 时，请始终使用显式 Intent，且不要为服务声明 Intent 过滤器。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 Intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 bindService()，系统会抛出异常。

### Intent 过滤器（IntentFilters）

Intent 过滤器是应用清单文件中的一个表达式，它指定该组件要接收的 Intent 类型。例如，通过为 Activity 声明 Intent 过滤器，您可以使其他应用能够直接使用某一特定类型的 Intent 启动 Activity。同样，如果您没有为 Activity 声明任何 Intent 过滤器，则 Activity 只能通过显式 Intent 启动。

### 构建 Intent

- 组件名称

  要启动的组件名称。

  ```java
  组件名称 Service.class
  Intent intent = new Intent(this,Service.class)
  ```

- 操作

  指定要执行的通用操作（例如，"查看"或"选取"）的字符串。

  ```java
  操作 ACTION_VIEW ACTION_EDIT ACTION_PICK
  intent.setAction(Intent.ACTION_VIEW);
  intent.setAction(Intent.ACTION_EDIT);
  intent.setAction(Intent.ACTION_PICK);
  ```

  ```xml
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.PICK" />
  <intent-filter>
  ```

- 数据 & 类别

  引用待操作数据和/或该数据 MIME 类型的 URI（Uri 对象）。提供的数据类型通常由 Intent 的操作决定。例如，如果操作是 ACTION_EDIT，则数据应包含待编辑文档的 URI。

  ```java
  数据：content://contacts/people 全部联系人  
  类别：text/plain 纯文本
  Uri data = Uri.parse("content://contacts/people");
  // intent.setData(data);
  // intent.setType("text/plain");
  intent.setDataAndType(data,"text/plain");
  ```

  > 警告：若要同时设置 URI 和 MIME 类型，请勿调用 setData() 和 setType()，因为它们会互相抵消彼此的值。请始终使用 setDataAndType() 同时设置 URI 和 MIME 类型。

  ```xml
  <intent-filter>
    <action android:name="android.intent.action.INSERT" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:scheme="content://contacts/people" />
    <data mimeType="text/plain" />
  </intent-filter>
  ```

- Extra

  携带完成请求操作所需的附加信息的键值对。正如某些操作使用特定类型的数据 URI 一样，有些操作也使用特定的附加数据。

  ```java
  Bundle bundle = new Bundle();
  bundle.putString("name", "jhon");
  intent.putExtra("key",bundle);
  ```

- 标志

  在 Intent 类中定义的、充当 Intent 元数据的标志

  ```java
  intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
  ```
