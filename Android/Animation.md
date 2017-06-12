---
title: Activity动画
date: 2016/9/9
tag:
  - Activity
urlname: Activity-Animation
---

# Activity Transition

Activity Transition是Material Design中提供的一种动画效果。它通过运动和切换不同状态之间的元素来产生各种动画效果。

**主要步骤**

1. 在Style主题中启用Activity Transition。
2. 定义Transition特效样式
3. 在Activity中代码实现上面两步（手动实现配置文件的效果）
4. 启动Activity

## 在Style主题中启用Activity Transition。

style.xml中添加如下属性

```xml
<!-- 允许使用transitions  必要* -->  
<item name="android:windowContentTransitions">true</item>
```

为Activity进入和推出指定transitions样式

```xml
<!-- 指定进入和退出transitions -->  
<item name="android:windowEnterTransition">@transition/explode</item>  
<item name="android:windowExitTransition">@transition/explode</item>
```

启动Activity方式为共享元素时 指定transitions样式

```xml
<!-- 指定shared element transitions -->  
<item name="android:windowSharedElementEnterTransition">  
    @transition/change_image_transform</item>  
<item name="android:windowSharedElementExitTransition">  
    @transition/change_image_transform</item>
```

## 定义Transition特效样式

根据需求在@transition目录下创建样式文件 如上面change_image_transform和explode 常用的Transition特效

```xml
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
    //进入和退出transitions效果
    <explode/>   //(爆裂) - 从场景中间移动视图进入或者退出
    <slide/>     //(滑动) - 视图从场景的一个边缘进入或者退出
    <fade/>      //(淡入淡出) - 从场景添加或者移除一个视图通过改变他的透明
    //共享元素过渡transitions效果(shared element)
    <changeBounds/>      // 改变目标视图的布局边界
    <changeTransform/>   // 裁剪目标视图边界
    <changeClipBounds/>  // 改变目标视图的缩放比例和旋转角度
    <changeImageTransform/>   // 改变目标图片的大小和缩放比例
</transitionSet>
```

## 在Activity中代码实现

手动在Activity用代码实现配置文件的效果

```java
// 允许使用transitions  
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);  

// 设置一个exit transition  
getWindow().setExitTransition(new Explode());  

//设置一个进入 transition
getWindow().setEnterTransition(new Fade());
```

## 启动Activity

当你已经设置了允许使用Transition并设置了Transition动画，你就可以通过ActivityOptions.makeSceneTransitionAnimation()方法启动一个新的Activity来激活这个Transition

**启动普通的Transition**

```java
startActivity(intent,ActivityOptions.makeSceneTransitionAnimation(this).toBundle());
```

**启用共享元素Transition**

在所有需要共享视图的Activity中，使用android:transitionName属性对于需要共享的元素分配一个通用的名字。

```java
Intent intent = new Intent(this, Activity2.class);  
//  shareView: 需要共享的视图  
// "shareName": 设置的android:transitionName="shareName"  
ActivityOptions options =
    ActivityOptions.makeSceneTransitionAnimation(this,shareView,"shareName");
startActivity(intent, options.toBundle());
```

如果有多个View需要共享，则通过Pair.create()方法创建多个匹配对然后传入ActivityOptions.makeSceneTransitionAnimation。

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(this,  
           Pair.create(view1, "agreedName1"),  
           Pair.create(view2, "agreedName2"));
```

如果不想使用transition可以设置options bundle为null。 当需要结束当前Activity并回退这个动画时调用Activity.finishAfterTransition()方法
