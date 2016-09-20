---
title: Activity
date: 2016/9/20
tag:
    - Android
    - Activity
    - 翻译
---


> 我该如何设计我的Android应用？我应该使用那种MVC模式？我应该使用事件总线（Event Bus）吗？

我们经常会看见开发人员询问的Android平台工程师，该使用何种设计模式和何种框架在他们的应用中。但是当我们回答时，通常很难又一个有用的建议或者都不算得上时一个建议。

（作者澄清：我在这使用“我们”代表Android工程师，而不是来代表Google和其他Android平台的团队。如何写出一个好的应用，Google或其他团队都提供了许多好的建议和意见。我也是推荐的）

你应该使用MVC？MVP？还是MVVM？，我不知道。相反，MVC是我在学校学到的，其他的选项我还必须使用Google搜索才能在这儿写下来。

这可能是令人惊讶的，因为Android像似使用Java语言API和更高层次的概念，有通用的规律保持应用编写，使用典型的应用框架保证应用正常工作。但在多数情况下，Android不是这样的。

Android应用调用一个Android核心API"system framework"可能会更好。多数情况下，Android API规定了如何与操作系统交互，但是任何只发生在Android应用内部的操作，Android API和这些都是不相关的。

这就是说，Android API在不同的操作系统上预计结果的不同（或需要更高权限），这可能导致在如何使用他们时混淆。

比如：让我们想一想，在常规系统中如何让操作系统定义“如何运行一个应用程序”，基本的约定是当如下代码运行时应用就该运行
```java
int main(...) {
   // 我的应用在这
}
```

所以当调用main()方法时，操作系统开始运行应用。应用的停止和他停止前所要执行的操作，应用如何设计都在这个main()方法中。 - 他仅仅是各开始

然而在Android中，我们明确规定了我们没有main()方法。因为我们需要这个平台在应用如何运行上更多的控制。特别是，我们想创建一个用户不需要去关心应用如何启动和停止的系统，而是系统去思考应用如何启动和停止。所以系统必须得到每一个应用程序内部发生什么的详细信息，并在需要这个应用时甚至他当前没有运行的情况下，用不同定义好的方式来启动应用。

要实现这样，我们分解了应用程序的典型主入口到几个不同类型，好让系统可以与他们交互，这几个不同的类型就是**Activity**、**BroadcastReceiver**、**Service**和**ContentProvider**。他们所有的都是关于应用怎样与系统进行交互（以及系统协调他们与其他应用程序交互）。当他们与系统交互时，我们是不用考虑他们是如何在应用内部实现的。

让我们简单地看一下这些API，他们在Android系统真正的意义：

#### Activity

这是一个应用和用户交互的入口，从系统的角度来看，他提供给应用程序的关键交互是：

* 跟踪那些用户当前关心的（屏幕上显示内容），并确保主机进程一直运行。
* 知道之前方法包含的内容可以使用户返回到之前方法（停止活动），从而保持这些进程优先级别。
* 当应用进程被杀死时，而用户返回到activity，他可以帮助应用恢复到之前状态。
* 由系统协调来提供应用程序实现彼此之间的用户操作（最典型的例子**分享**功能）。

我们不用关心：

一旦我们进入你的用户界面，我们完全不关心你内部如何组成。
Once we have gotten in to this entry-point to your UI, we really don't care how you organize the flow inside.  Make it all one activity with  manual changes to its views, use fragments (a convenience framework we provide) or some other framework, or split it into additional internal activities.  Or do all three as needed.  As long as you are following the high-level contact of activity (it launches in the proper state, and saves/restores in the current state), it doesn't matter to the system.


BroadcastReceiver

This is a mechanism for the system to deliver events to the application that may be outside of a regular user flow.  Most importantly, because this is another well-defined entry into the app, the system can deliver broadcasts to apps even if they aren't currently running.  So, for example, an app can schedule an alarm to post a notification to tell the user about an upcoming event...  and by delivering that alarm to a BroadcastReceiver of the app, there is no need for the app to remain running until the alarm goes off.

What we don't care about:

Dispatching events within an app is an entirely different thing.  Whether you use some event bus framework, implement your own callback system, whatever...  there is no reason to use the system's broadcasting mechanism, since you aren't dispatching events across apps.  (In fact there is good reason not to -- there is a lot of unnecessary overhead and many potential security issues if using a global broadcast mechanism for the internal implementation of an app.)  We do provide the LocalBroadcastManager convenience class that implements a purely in-process intent dispatching system with a similar API to the system's APIs, if you happen to like them.  But again, there is no reason to use that over something else for things going on purely within your app.

Service

A general-purpose entry point for keeping an app running in the background for all kinds of reasons.  There are actually two very distinct semantics services tell the system about how to manage an app:

Started services are simply telling the system to, for some reason, "keep me running until I say I am done."  This could be to sync some data in the background or play music even after the user leaves the app.  Those also represent two different types of started services that modify how the system handles them:

• Music playback is something the user is directly aware of, so the app tells the system this by saying it wants to be foreground with a notification to tell the user about it; in this case the system knows that it should try really hard to keep that service's process running, because the user will be unhappy if it goes away.

• A regular background service is not something the user is directly aware as running, so the system has more freedom in managing its process.  It may allow it to be killed (and then restarting the service sometime later) if it needs RAM for things that are of more immediate concern to the user.

Bound services are running because some other app (or the system) has said that it wants to make use of the service.  This is basically the service providing an API to another process.  The system thus knows there is a dependency between these processes, so if process A is bound to a service in process B, it knows that it needs to keep process B (and its service) running for A.  Further, if process A is something the user cares about, than it also knows to treat process B as something the user also cares about.

Because of their flexibility (for better or worse), services have turned out to be a really useful building block for all kinds of higher-level system concepts.  Live wallpapers, notification listeners, screen savers, input methods, accessibility services, and many other core system features are all built as services that applications implement and the system binds to when they should be running.

What we don't care about:

Android doesn't care about things going on within your app that don't have any impact on how it should manage your process, so there is no reason to use services in these cases.  For example, if you want to start some background operation to download data for your UI, you should not use a service for this -- it is actually important to not be telling the system to keep your process running while doing this, because it really doesn't need to be and the system would be better off having more freedom in managing it with other things the user is doing.

If you just make a simple background thread (or whatever non-service mechanism you want) to do the downloading, you will get the semantics you want: while the user is in your UI, the system will keep your process running for that, so the download will never be interrupted.  When they leave your UI, your process will still be kept around (cached) and able to continue downloading, as long as its RAM isn't needed elsewhere.

Likewise for connecting different parts of your app together, there is no reason to bind to a service that is running in the same process as the one binding to it.  Doing so is not actively harmful -- the system just sees a dependency from the process to itself so doesn't try to keep it around any more than usual -- but it is a bunch of unnecessary work for both you and the system.  Instead, you can just use singletons or other normal in-process patterns for connecting pieces of your app together.

ContentProvider

Finally, the ContentProvider is a fairly specialized facility for publishing data from an app to other places.  People generally think of them as an abstraction on a database, because there is a lot of API and support built in to them for that common case...  but from the system design perspective, that isn't their point.

What these are to the system is an entry-point into an app for publishing named data items, identified by a URI scheme.  Thus an app can decide how it wants to map the data it contains to a URI namespace, handing out those URIs to other entities which can in turn use them to access the data.  There are a few particular things this allows the system to do in managing an app:

• Handing out a URI doesn't require the app remain running, so these can go all over the place with the owning app being dead.  Only at the point where someone tells the system, "hey give me the data for this URI" does it need to make sure the app owning that data is running, so it can ask the app to retrieve and return the data.

• These URIs also provide an important fine-grained security model.  For example, an application can place the URI for an image it has on the clipboard, but leave its content provider locked up so nobody can freely access it.  When another app pulls that URI off the clipboard, the system can give it a temporary "URI permission grant" so that it is allowed to access the data only behind that URI, but nothing else in the app.

What we don't care about:

It doesn't really matter how you implement the data management behind a content provider; if you don't need structured data in a SQLite database, don't use SQLite.  For example, the FileProvider helper class is an easy way to make raw files in your app available through a content provider.

Also, if you are not publishing data from your app for others to use, there is no need to use a content provider at all.  It is true, because of the various helpers built around content providers, this can be an easy way to put data in a SQLite database and use it to populate UI elements like a ListView.  But if any of this stuff makes what you are trying to do more difficult, then feel free to not use it and instead use a more appropriate data model for your app.﻿
