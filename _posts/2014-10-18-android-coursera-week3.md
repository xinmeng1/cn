---
layout: post
title: Coursera Android公开课 Week3 笔记(Intent, Permission, Fragment)
categories: [Andorid]
tags: [Coursera, 公开课, Android]
---

## 概述

Week3的内容主要是介绍 Intent, Permission和Fragment. 在这里根据授课内容对其中的要点进行总结阐述, 并且关键代码展示出来. 最后把作业中关键问题进行详细讨论, 最后对Quiz的内容进行简要分析.

建议首先学习Coursera公开课内容, 本文在学习完之后再进行详细讨论,如果有任何问题可以留言.

## Intent详解

### Intent类

#### 简介

Intent类是一种数据结构:
 
1. 表示一种将要被执行的操作, 
2. 表示一种将要发生的事件.

例如用Intent在启动Activity时,就是表示将要启动的Activity. 本节主要讲解第一种用途. 在BroadcastReceivers讲解时再涉及事件通知.

Intents 提供灵活的语言来指定将要执行的操作.例如,选取联系人, 照相,以及输入电话号码等. 

Intent 是在准备执行某些任务的组件中进行构造, 然后由指定的Activity接受并执行该任务.

####Intent Fields 域

Intent构造包括如下内容:

#####**1.Action**

准备执行操作的描述字符串,例如

	ACTION_DIAL – Dial a number 
	ACTION_EDIT – Display data to edit 
	ACTION_SYNC – Synchronize device data with server 
	ACTION_MAIN – Start as initial activity of app 

**如何配置 Action?**

	Intent newInt= new
	Intent(Intent.ACTION_DIAL);
OR

	Intent newInt= new Intent();
	newInt.setAction(Intent.ACTION_DIAL);

#####**2.Data**

Data associated with the Intent Formatted as a Uniform Resource 
Identifier (URI) 

表示Intent附带的数据, 被格式化为URI形式.

地图显示数据 

	Uri.parse(“geo:0,0?q=1600+Pennsylvania+Ave+Washington+DC”)

在电话面板上将要输入的数字
 
	Uri.parse(“tel:+15555555555”)

**配置Data?**

	Intent newInt= new Intent (Intent.ACTION_DIAL,
	Uri.parse("tel:+15555555555"));
Or 

	Intent newInt= new Intent(Intent.ACTION_DIAL);
	newInt.setData(Uri.parse("tel:+15555555555"));

#####**3.Category**

Additional information about the components that can handle the intent 
能够控制该intent的附加信息.

能够被浏览器调用来通过URI显示数据

	Category_browsable
能够被初始化任务的activity或者能够在顶层app启动器列表中显示	
	
	Category_launcher

#####**4.Type**

Specifies the MIME type of the Intent data 

	image/*, image/png, image/jpeg
	text/html, text/plain
如果没有指定, Android会隐含推断类型.

**如何配置?**

	Intent.setType(String type)
 
 or
 
	Intent.setDataAndType(Uri data,String type)

#####**5.Component**
The component that should receive this intent 

Use this when there’s exactly one component that should receive the 
intent

接收该intent的组件, 当希望准确指定接收intent的组件时使用.

**如何配置**

	Intent newInt= Intent(Context packageContext, Class<?> cls);
	Intent newInt= new Intent ();
	setComponent(); 
	setClass();
	setClassName();
 
#####**6.Extras**

Add’l information associated with Intent Treated as a map (key-value pairs) 

为Intent增加而外信息, 并使用map来进行表示(key-value)

Intent.EXTRA_EMAIL: email recipients 
	
	Intent newInt= new Intent(Intent.ACTION_SEND);
	newInt.putExtra(android.content.Intent.EXTRA_EMAIL, 
	new String[]{
		“aporter@cs.umd.edu”, “ceo@microsoft.com”,
		“potus@whitehouse.gov”,“mozart@musician.org”
		}
	);

**如何配置?**

	putExtra(String name, String value);
	putExtra(String name, float[] value);

#####**7.Flags**

Specify how Intent should be handled

指明如何操作Intent

	FLAG_ACTIVITY_NO_HISTORY
表示不用把Activity放到堆栈历史中 

	FLAG_DEBUG_LOG_RESOLUTION
打印出额外的日志信息当Intent被处理时

**如何配置?**

	Intent newInt= new Intent(Intent.ACTION_SEND);
	newInt.setFlags(Intent.FLAG_ACTIVITY_NO_HISTORY);

### 启动Activity
 
前面介绍了一堆内容, 这里简单了解一下使用Intent启动Activity, Intent可以翻译为"意图", 本课程主要讲解如何使用Intent进行启动Activity. 

这里启动Activity分为两类:

1.  Explicit  Activation 显示激活
2.  Implicit Activation (via Intent Resolution) **隐式激活**

显示激活是指我们明确知道激活启动的Activity, 这样可以直接进行激活.

隐式激活是指我们通过一些参数激活一些现有的Activity, 例如联系人, 地图以及其他程序的Activity. (隐式调用是重点 ★)

启动Activity的命令,其目标就是以上两种方式指明:
	
	startActivity(Intent intent,…)
	startActivityForResult(Intent intent, …) //带返回值

第一种显示调用,案例比较简单,通过new一个Intent,第一个参数为`当前Activity.this`, 第二个参数为`目标Activity.class`,最后启动Intent就OK了.

	// Create an explicit Intent for starting the HelloAndroid Activity 
	Intent helloAndroidIntent = new  Intent(LoginScreen.this, HelloAndroid.class);				
	// Use the Intent to start the HelloAndroid Activity 
	startActivity(helloAndroidIntent);

重点介绍第二种隐式调用Intent.

如果你要启动的Activity没有被显示指定, 那么Android会尝试激活那些匹配的Activities, 而这个过程就叫做 **Intent Resolution**, 下面就介绍下 Intent Resolution的过程:

1. Intent被描述为一个将要执行的操作
2. IntentFilters 用来描述那些操作 Activity能够处理, 该属性在 Android的Manifest.xml文件中指定, 当然也可以程序动态添加. 也就是说如果在Activity的描述文件中加入这个节点,就表示该Activity可以处理默认的内容, 当然这个描述完全可以自定义, 譬如作业中的例子(见后面)

IntentFilter 案例, 表示该Activity能够处理 DIAL 操作, 也就是处理打电话.

	<activity …> 
    <intent-filter …>
    …
    <action android:name="android.intent.action.DIAL" />
    <data android:scheme=”geo" /> //处理地位位置
    <category android:name="string" /> //处理的类别
	…
    …
    </intent-filter>  
    
    …
    </activity> 

具体可参考官方文档 [intents-filters](developer.android.com/guide/components/intents-filters.html)

最后以一个Map Application的案例作为介绍, 深入理解 intent-filter
	
	<intent-filter …>
		<action android:name= 
		"android.intent.action.VIEW" />
		<category android:name= 
		"android.intent.category.DEFAULT" />
		<category android:name=
		"android.intent.category.BROWSABLE”/>
		<data android:scheme= "geo”/>
	</intent-filter>
 
**注意: 如果希望介绍implicit intent, Activity应该需要制定Intent-Filter的Category为 `"android.intent.category.DEFAULT”`**

当然如果多个Activity都制定同样的Intent, 那么需要 Priority来进行区分. 其值在 -1000- 1000 之间. 数值越大, 其优先权越高.

	% adb shell dumpsys package

Investigate Intent Filter. 

dumpsys简单介绍：该命令用户打印出当前系统信息，默认打印出所有service信息，可以在命令后面加入activity参数，只打印出activity相关的信息。


###回传参数

这部分是在作业中遇到的, 表示通过Intent启动一个Activity后, 可以通过该方法将启动的Activity中某些值传回到当前的Activity, 

具体方法参见[Result](http://developer.android.com/training/basics/intents/result.html)

A1调用A2, 返回d2到A1, 其主要步骤
1. A1 定义 GET_TEXT_REQUEST_CODE
2. A1 startActivityForResult(A2,GET_TEXT_REQUEST_CODE);
3. A2 准备d2, String returntext= mEditText.getText().toString();
4. A2 Intent中加入 d2 :loaderintent.putExtra("name", returntext);
5. A2 Intent中设置返回状态: setResult(RESULT_OK,loaderintent);
6. A1 实现 onActivityResult(int requestCode, int resultCode, Intent data); 其中requestCode == GET_TEXT_REQUEST_CODE, resultCode ==A2中返回状态, data==A2中返回的数据.
7. 数据位map类型, 即string name - value (e.g name--> returntext)

## Permission详解

###简介

Andorid权限是为了保护资源和数据. 也就是说限制访问某些资源, 例如用户信息(contacts), 资费敏感API调用(SMS/MMS), 系统资源使用(摄像头).

Permission在manifest.xml中使用string进行声明, 声明的Permission包括两种, 一种是自己使用的权限, 另一种是其他组件调用所需要的权限.

###Permission使用

#### 申请使用现有的权限

`<uses-permission>`用来指定权限, 如果程序中加入这个选项, 在安装的时候需要用户同意授予该权限.

	<manifest … > 
	…
	<uses-permission android:name=
	"android.permission.CAMERA”/>
	<uses-permission android:name=
	"android.permission.INTERNET”/>
	<uses-permission android:name=
	“android.permission.ACCESS_FINE_LOCATION”/>
	…
	</manifest >

#### 定义并授予权限

该使用情景是假设你的APP需要执行一些隐私或者危险的操作,所以你不希望其他apps简单的调用你的应用, 你就可以在应用中加入自定义的权限.(define&enforce)

举例说明, A1是一个危险程序, 需要限制它被调用, 但是这时程序B1需要调用A1, 这样B1需要A1的调用权限, 下面是实现过程:

1.A1定义权限, 使用一下代码放在manifest根节点中
		
	<!-- Defines a custom permission -->
	<permission
	android:name="course.examples.permissionexample.BOOM_PERM"
	android:description="@string/boom_perm_string"
	android:label="@string/boom_permission_label_string" >
	</permission>

2.B1的manifes根节点中申请A1中的权限

	<!--  grant the "...BOOM_PERM permission to this application -->
	<uses-permission android:name="course.examples.permissionexample.BOOM_PERM" />


3.A1中manifest中加入intent-filter

	<intent-filter>
    <action android:name="course.examples.permissionexample.boom.boom_action" />
    <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>

4.B1中使用intent调用A1

	// String used to represent the dangerous operation
	private static final String ACTION_BOOM = "course.examples.permissionexample.boom.boom_action";
	// Launch an Activity that can receive the Intent using Activity.startActivity()  
	startActivity(new Intent(ACTION_BOOM));


###Component Permissions 组件权限

独立的Component也可以设置自己的权限, 这样限制其他Component对它的访问, 


Component permissions take precedence over application-level permissions 

而且Component权限优先于Application级别的权限.

###Activity Permission 活动权限

该权限可以限制那些组件能够启动Activity. 一下方法执行时可以检查是否拥有权限, 如果没有被授予权限, 就会抛出 SecurityException

	startActivity()
	startActivityForResult()

###Service Permission 服务权限

该权限限制服务的访问, 在一下方法执行是检查权限, 同样没有授予权限就会抛出异常.

	Context.startService()
	Context.stopService() 
	Context.bindService()

###BroadcastReceiver Permission 广播权限

功能同上述的权限, 权限检查会在多处进行.

###ContentProvider Permission 内容提供权限

功能同上述的权限, 权限检查会在多处进行.

## Fragment详解

### 简介

现在的平板电脑都有很大的屏幕, 它们能够支持多重的UI Panes以及用户的操作. 也就是说需要充分利用屏幕空间, 使用fragment(碎片)来实现大屏幕的多片分割. 

Fragment的作用:
1. 能够在Activity中展示行为和部分UI
2. 多个Fragment能够嵌入到Activity中创造多窗口UI
3. 单个Fragment能够在多个Activity中进行复用

### Fragment生命周期

![enter image description here](https://lh6.googleusercontent.com/-4lGvQ0pltLU/VELhOFUjXgI/AAAAAAAAAe0/nHE8iWU-tDo/s0/fragment-lifecycle.png "fragment-lifecycle.png")

1. onAttach(): Fragment首次和Activity连接
2. onCreate(): 初始化Fragment
2. onCreateView(): Fragment设置并返回用户接口
2. onActivityCreated(): Activity完成onCreate(), Fragment已经被安装
2. onStart(): 所属的Activity可见
2. onResume(): 所属的Activity可见并且准备显示用户接口
2. onPause(): 所属的Activity可见, 但是没有获得焦点
2. onStop(): 所属的Activity不在可见
2. onDestroyView(): 之前在onCreateView()创建的视图在这里与Activity解除连接2. (clean up view resources)
2. onDestroy(): Fragment不再可用(clean up fragment resources)
2. onDetach(): Fragment不再和Activity有关联(Null out references to hosting Activity)

###Fragment的使用

#### 加入Fragment到Activity

1. Declare it statically in the Activity’s layout file 直接在layout文件中说明
2. Add it programmatically using the fragmentManager 动态的使用FragmentManager加入

#####第一种方法:

 Layout在 onCreateView()中进行填充和实现Fragment, onCreateView()必须返回Fragment的Layout的根节点. 

布局文件: 

    <fragment
        android:id="@+id/titles"
        android:layout_width="0px"
        android:layout_height="match_parent"
        android:layout_weight="1"
	class="course.examples.Fragments.StaticLayout.TitlesFragment" />

Activity的onCreate():


		// Get the string arrays with the titles and qutoes
		mTitleArray = getResources().getStringArray(R.array.Titles);
		mQuoteArray = getResources().getStringArray(R.array.Quotes);
		
		setContentView(R.layout.main);
		
		// Get a reference to the QuotesFragment
		mDetailsFragment = (QuotesFragment) getFragmentManager()
				.findFragmentById(R.id.details);

#####第二种方法: 
 
 1. 获取FragmentManager的参考
 2. 创建一个FragmentTransaction
 3. 添加Fragment
 4. 提交FragmentTransaction

只需要四步就可以完成Fragment的添加.

#### Fragment动态适用屏幕

根据作业内容来学习如何用Fragment来适应不同的屏幕显示内容.

1.创建两种布局文件, layout和layout-large文件夹中, layout下屏幕不使用单Fragment布局,而大屏幕可以使用多Fragment布局.
2.程序处理中加入函数 `!isInTwoPaneMode()` 用来判断是否应该加入多Fragments,使用`findViewById(R.id.fragment_container) == null;`来实现, 也就是layout中有一个Fragment_container, 而layout-large中没有, 而这个container就是用来动态填充Fragment的.
3. 如果是小屏幕, 则动态加入Fragment到container, 使用如下步骤, 参考上一节.
 
        FragmentManager fragmentManager = getFragmentManager();
		FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
		fragmentTransaction.add(R.id.fragment_container,mFriendsFragment);
		fragmentTransaction.commit();

###Fragment的配置改变

如果调用`setRetainInstance(true)`, 那么Android将不会销毁Fragment. 

详细说明setRetainInstance()方法, 此方法可以有效地提高系统的运行效率，对流畅性要求较高的应用可以适当采用此方法进行设置。

Fragment有一个非常强大的功能——就是可以在Activity重新创建时可以不完全销毁Fragment，以便Fragment可以恢复。在onCreate()方法中调用`setRetainInstance(true/false)`方法是最佳位置。当在`onCreate()`方法中调用了setRetainInstance(true)后，Fragment恢复时会跳过`onCreate()`和`onDestroy()`方法，因此不能在onCreate()中放置一些初始化逻辑.

[setRetainInstance中文参考](http://blog.csdn.net/weihan1314/article/details/7997421)

## 作业重点问题详解

###Intent 作业

####1.显式调用很简单

		// TODO - Create a new intent to launch the ExplicitlyLoadedActivity class
		Intent explicitIntent = new Intent(ActivityLoaderActivity.this, ExplicitlyLoadedActivity.class);
		// TODO - Start an Activity using that intent and the request code defined above
	startActivityForResult(explicitIntent,GET_TEXT_REQUEST_CODE);

####2.显示调用回传值

参见上面的步骤也比较容易, 关键是几个参数的设置.

####3.隐式调用

这里涉及如何调用 Intent choose 来选择合适的程序处理 Intent.

A1 调用网页x, 在浏览器B1和自定义的程序B2显示

1. A1定义 `Intent baseIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"));`
2. A1定义 `Intent chooserIntent = Intent.createChooser(baseIntent, (CharSequence)title);` 用于选择应用
3. B2 程序的 menifest 需要加入 处理网页的intent-filter `<action android:name="android.intent.action.VIEW"/>`
4. A1 启动intent baseIntent 

###Permission 作业

和案例的程序基本一致,但是需要注意一个问题, 前面已经提到了就是在intent-filter中需要加入下面category节点, 才能被其他程序调用

	<category android:name="android.intent.category.DEFAULT" />

###Fragment作业

比较简单, 直接就是Fragment管理方法, 一个是添加,一个是替换

## Quiz问题详解
