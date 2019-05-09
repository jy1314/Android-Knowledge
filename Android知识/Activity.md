学Android就绕不开四大组件，那么说到四大组件，必然就要提到四大组件之首的Activity。
实际应用中，我们接触的最多的就是Activity，那么今天就来仔细的研究一下它。
本文的要点如下：
* Activity简介
* Manifest配置文件
* Activity生命周期
* Activity的启动模式

## 1、Activity简介
相信学过Android的人最先接触到的就是Activity，它是一种可以**包含用户界面**的组件，主要用于**和用户交互**。在一个应用程序中可以包含多个Activity。

## 2、Manifest配置文件
AndroidManifest.xml是整个应用的主配置清单文件，包括应用的包名、版本号、组件、权限等信息，它用来记录应用的相关的配置信息。
一个简单的AndroidManifest.xml如下：
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.retrofit2">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>
```

#### manifest部分：
manifest是AndroidManifest.xml配置文件的根标签， 必须指定xlmns:android和package属性， 且只包含一个application节点。
**xlmns:android**指定了Android的命名空间，默认情况下是http://schemas.android.com/apk/res/android。
**package**是标准的应用包名，也是一个应用进程的默认名称，为避免命名空间的冲突，一般会以应用的域名来作为包名。一般情况这部分不需要做改动。
#### application标签:
在application标签下有几个面向全局的属性：android:icon（图标）、android:label（标题）、android:theme（主题样式）。
而在application标签里面包裹着安卓四大组件：activity（活动）、service（服务）、content provider（内容提供者）以及broadcast receiver（广播接收器）。其中除了广播接收器可以动态注册外，其他三大组件都必须在application里注册才能使用。在这四个组件添加到application时，一定要声明android:name属性，值以包名.类名的形式，其中包名（package）可省写成.类名即可。
#### Activity标签：
application标签下的Activity标签声明了Activity的启动方式以及是否是LAUNCHER即程序入口。
#### uses-permission标签:
可用uses-permission标签声明一系列系统权限，需要的时候添加就可以。

## 3、Activity的生命周期
先上个经典的图：
![Activity生命周期](https://upload-images.jianshu.io/upload_images/17755742-f5faed6ea8fb2350.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这张图相信大家在初学android的时候肯定都见过，那么我们下面就来仔细聊聊这个图的内容。
在正常情况下，一个Activity从启动到结束会以如下顺序经历整个生命周期：
onCreate()->onStart()->onResume()->onPause()->onStop()->onDestory()。包含了六个部分，还剩一个onRestart()没有调用。
**那么每一部分具体是什么用处呢？**
> * onCreate()表示Activity 正在**创建**，常做初始化工作，如setContentView界面资源、初始化数据；
>* onStart()表示Activity 正在启动，这时Activity **可见但不在前台**，无法和用户交互；
>* onResume()表示Activity 获得焦点，此时Activity **可见且在前台并开始活动**
>* onPause()表示Activity 暂停，可做数据存储、停止动画等操作
>* onStop()表示activity 即将停止，可做稍微重量级**回收工作**，如取消网络连接、注销广播接收器等
>* onDestroy()表示Activity 即将销毁，常做**回收工作、资源释放**
>* 当Activity由后台切换到前台，由**不可见到可见**时会调用onRestart()，表示Activity 重新启动。

**onStart()和onResume()/onPause()和onStop()的区别**
>onStart()与onStop()是从Activity是否可见这个角度调用的，onResume()和onPause()是从Activity是否显示在前台这个角度来回调的，在实际使用没其他明显区别。

**Activity A启动另一个Activity B会回调哪些方法**
>既然onStart()与onStop()是从Activity是否可见这个角度调用的，那么Activity A启动另一个Activity B回调的方法的顺序就应该是：Activity A的onPause() -->Activity B的onCreate()-->onStart()-->onResume()-->Activity A的onStop()；如果Activity B是完全透明的或是对话框Activity，则最后不会调用Activity A的onStop()。

**Activity的异常恢复**
>当非人为终止Activity时，比如横竖屏切换、系统配置发生改变时导致Activity被杀死并重新创建、资源内存不足导致低优先级的Activity被杀死，会调用 onSavaInstanceState() 来保存状态，该方法调用在onStop之前。
那么就有疑问了，**为什么不直接用onPause()方法来保存状态呢？**
因为onSaveInstanceState()只在Activity异常退出时才会调用，适用于对临时性状态的保存，而onPause()适用于对数据的持久化保存。
**之后具体要怎么恢复呢？**
异常关闭的Activity被重新创建时会调用onRestoreInstanceState（该方法在onStart之后），并将onSavaInstanceState保存的Bundle对象作为参数传到onRestoreInstanceState与onCreate方法。因此可通过**onRestoreInstanceState(Bundle savedInstanceState)**和**onCreate(Bundle savedInstanceState)**来判断Activity是否被重建，并取出数据进行恢复。但需要注意的是，onCreate方法是无论是否异常退出都会调用的，因此在onCreate中的savedInstanceState是可能为空的，所以在onCreate取出数据时一定要先判断savedInstanceState是否为空。另外，谷歌更推荐使用onRestoreInstanceState进行数据恢复，因为这个这个方法的意义所在。

**如何避免横竖屏切换Activity销毁和重建**
>可以通过在AndroidManifest文件的Activity中指定如下属性:
>```
>android:configChanges = "orientation| screenSize"
>```
>这样的话，横竖屏切换时就会回调 onConfigurationChanged方法：
>```
>@Override
>   public void onConfigurationChanged(Configuration newConfig) {
>        super.onConfigurationChanged(newConfig);
>    }
>```


## 4、Activity的启动模式
相信大家都知道，Activity有四种启动模式。Activity的管理是采用任务栈的形式，任务栈采用“后进先出”的栈结构。
>**Standard标准模式**：每次启动一个Activity就会创建一个新的实例。
**SingleTop栈顶复用模式**：如果新Activity已经位于任务栈的栈顶，就不会重新创建，并回调 onNewIntent(intent) 方法。
**SingleTask栈内复用模式**：只要该Activity在一个任务栈中存在，都不会重新创建，并回调 onNewIntent(intent) 方法。如果不存在，系统会先寻找是否存在需要的栈，如果不存在该栈，就创建一个任务栈，并把该Activity放进去；如果存在，就会创建到已经存在的栈中。
**SingleInstance单实例模式**：具有此模式的Activity只能单独位于一个任务栈中，且此任务栈中只有唯一一个实例。

**为什么要有这四种启动模式呢？都用Standard有什么不好么？**
先说SingleTop，在通知栏点击收到的通知，然后需要启动一个Activity，这个Activity就可以用singleTop，否则每次点击都会新建一个Activity。另外，在某个场景下连续快速点击，如果是Standard模式，就会启动了两个Activity，而singleTop则不会。
再说SingleTask，对于大部分应用，当我们在主界面点击回退按钮的时候都是退出应用，那么当我们第一次进入主界面之后，主界面位于栈底，以后不管我们打开了多少个Activity，只要我们再次回到主界面，都应该使用将主界面Activity上所有的Activity移除的方式来让主界面Activity处于栈顶，而不是往栈顶新加一个主界面Activity的实例，通过这种方式能够保证退出应用时所有的Activity都能报销毁。
最后是SingleInstance，这种模式的使用情况就比较罕见了，呼叫来电界面是用了这种方式。

**具体使用：**
启动模式有2种设置方式：
1. 在AndroidMainifest设置；


```
<activity android:launchMode="启动模式"
//属性
//standard：标准模式
//singleTop：栈顶复用模式
//singleTask：栈内复用模式
//singleInstance：单例模式
//如不设置，Activity的启动模式默认为**标准模式（standard）**
</activity>
```

2. 通过Intent设置标志位。

```
Intent inten = new Intent (ActivityA.this,ActivityB.class);
intent,addFlags(Intent,FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

标记位|启动模式
--|--
FLAG_ACTIVITY_SINGLE_TOP|SingleTop
FLAG_ACTIVITY_NEW_TASK|SingleTask
FLAG_ACTIVITY_CLEAR_TOP|所有位于其上层的Activity都要移除，SingleTask模式默认具有此标记效果
FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS|具有该标记的Activity不会出现在历史Activity的列表中，即无法通过历史列表回到该Activity上
区别：
* 优先级不同
Intent设置方式的优先级 > Manifest设置方式，即 以前者为准
* 限定范围不同
Manifest设置方式无法设定 FLAG_ACTIVITY_CLEAR_TOP；Intent设置方式无法设置单例模式（SingleInstance）




