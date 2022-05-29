# Android 学习笔记

【wangs7】2021/08/12

[TOC]

# 1 Android 项目结构简介

【主要内容】Android 项目结构简介。

## 分析简单的Android程序

1. `.gradle` 和 `.idea` ：这两个目录是放置 Android Studio 自动生成的一些文件。

2. `app` 项目中的代码、资源等内容几乎都在这个目录下。

   > 1. build：包含编译时自动生成的文件。
   > 2. libs：第三方的jar包需要放到该目录下，该目录下的jar包都会被自动添加到构建路径里去。
   > 3. androidTest：用来编写Android Test测试用例的，可以对项目进行一些自动化测试。
   > 4. java：放置所有Java代码的地方。
   > 5. res：项目中使用到的所有图片、布局、字符串等资源都要存放在这个目录下。
   >    * 图片放在 drawable
   >    * 布局放在 layout
   >    * 字符串放在 values 等 
   >    * [详细介绍](#res)
   > 6. AndroidManifest.xml：整个Android项目的配置文件，程序中所有的四大组件都需要在这个文件里注册，另外还可以在这个文件中给应用程序添加权限声明。
   > 7. test：用来编写Unit Test测试用例的，对项目自动化测试的另一种方式。
   > 8. build.gradle：这是app模块的gradle构建脚本，这个文件中会指定很多项目构建相关的配置。
   > 9. proguard-rules.pro：这个文件用于指定项目代码的混淆规则。

3. `build` ：包含编译时自动生成的文件。

4. `gradle` ：含有gradle wrapper的配置文件，设置方式**File -> Settings -> Build,Execution,Deployment -> Gradle** 。

5. `build.gradle` ：项目全局的==gradle构建脚本==。

6. `gradle.properties` ：全局的==gradle配置文件==，在这里配置的属性会影响到项目中所有的gradle编译脚本。

7. `gradlew*` ：用命令行执行gradle需要的文件。

8. 其它 略。 



## 简析运行机制

新建一个Android项目，选择 Empty Activity ，然后都按默认选择，生成虚拟环境运行，能看到一个页面上有 “Hello World!” 的文字。

首先打开 AndroidManifest.xml 文件

```xml
<activity android:name=".HelloWorldActivity">
    <intent-filter>
        <!-- 表示该项目的主活动 在手机上点击图标最先启动这个活动 -->
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

这段表示对HelloWorldActivity这个活动进行注册，没有在AndroidManifest.xml里注册的活动是不能够使用的。

活动是Android应用程序的门面，所有在应用中能看到的东西，都是在活动中的。HelloWorldActivity.java，代码如下：HelloWorldActivity.java

```java
//HelloWorldActivity 继承 AppCompatActivity 向下兼容 Activity
public class HelloWorldActivity extends AppCompatActivity {
    @Override //重写 onCreate 活动执行时必须被调用的方法
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);       
        /* 给当前活动引入hello_world_layout布局 */
        setContentView(R.layout.hello_world_layout);
    }
}
```

Android程序设计讲究逻辑和视图分离，因此时不推荐在活动中直接编写界面的，更加通用的一种做法是在布局文件中编写界面，然后活动中引入进来。`HelloWorldActivity`类中有一个`setContentView`方法引入了一个布局`hello_world_layout`，布局文件都定义在`res/layout`目录下的，在该目录中找到 hello_world_layout.xml 这个文件。hello_world_layout.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/hello_world_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimeo/activity_vertical-margin"
    android:paddingLeft="@dimeo/activity_horizontal_margin"
    android:paddingRight="@dimeo/activity_horizontal_margin"
    android:paddingTop="@dimeo/activity_vertical-margin"
	tools:cotext="come.example.helloworld.HelloWorldActivity">
	
    <!-- 系统控件 用于布局显示文字 -->
    <TextView  
    	android:layout_width="match_parent"
        android:layout_height="match_parent"
		android:text="Hello World!" />

</RelativeLayout>

```

控件 TextView 中的 `android:text="Hello World!"` 用于显示 Hello World！ 的文字，目前就是这个简单程序运行的机制。



## <span id="res">项目中的资源 res/</span>

项目中的资源基本全部都在 `./app/res/` 中，res中有很多文件夹用于各种文件的分类

>res
>
>├ drawable*/		**图片**
>
>├ mipmap*/		**图标**
>
>├ values*/		**字符串、样式、颜色等配置**
>
>├ layout*/		**布局文件**
>
>└ others  ...



drawable开头的文件夹都是用来放图片的；mipmap开头的是文件夹是用来放图标的；values开头的是用来放字符串、样式、颜色等配置的；layout开头的是用来放布局文件的。对于文件夹的“后缀”的说明，有时候我们需要不同分辨率的图片供程序恩局不同设备和运行情况自动加载，这时我们就需要不同的drawable文件夹，`drawable` `drawable-hdpi` `drawable-xhdpi` `drawable-xxhdpi` 等文件夹，其它的文件夹也类似。

项目资源的使用

打开 `res/values/strings.xml` 文件

```xml
<resource>
	<string name="app_name">HelloWorld</string>
</resource>
```

可以看到这里定义了了一个程序名的字符串，我们有以下两种方式引用它。

* 在代码中通过 `R.string.hello_world` 可以获得该字符串得引用。
* 在 XML 文件中通过 `@string/hello_world` 可以获得引用。

其中 `string` 是可以替换的，如果是图片资源就替换成 `drawable` ，如果是布局资源就替换成 `layout` ，以此类推。打开文件 AndroidManifest.xml 文件。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.helloworld">

    <application
        tools:replace="android:label,android:theme,android:icon"
        android:allowBackup="true" 
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" >
    </application>

</manifest>
```

其中 HelloWorld 项目的应用图标就是通过 `android:icon="@mipmap/ic_launcher"` 来指定的，应用名称是通过 `android:label="@string/app_name"` 来指定的。



## build.gradle 文件

HelloWorld有两个 build.gradle 文件，`./build.gradle` 和 `./app/build.gradle` ,先来看最外层的 build.gradle。

./build.gradle

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: "config.gradle"
buildscript {
    repositories {
        jcenter() //代码托管仓库 声明后可引用仓库资源
        google()
    }
    dependencies {
        //声明 插件以及版本号
        classpath 'com.android.tools.build:gradle:4.2.2'
    }
}
allprojects {
    repositories {
        jcenter()
        google()
    }
}
task clean(type: Delete) {
    delete rootProject.buildDir
}

```

./app/build.gradle

```groovy
//应用了一个插件 有两种值可选
//com.android.application 表示这是一个应用程序模块
//com.android.library 表示这是一个库模块
apply plugins: 'com.android.application'

android {
    //指定项目编译器版本
    compileSdkVersion 30
	//buildToolsVersion "xx.x.x" 用于执行项目构建工具的版本
    
    //对项目的更多细节进行配置
    defaultConfig {
        //指定包名
        applicationId "com.example.helloworld"
        //最低SDK版本
        minSdkVersion 24
        //目前标值版本已经充分测试
        targetSdkVersion 30
        //项目版本号
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
			//是否对代码进行混淆
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

//可以指定当前项目的所有依赖
dependencies {
    //将 libs 目录下的所有 jar 文件都添加到项目构建路径中
	implementation fileTree(dir: 'libs', include: ['*.jar'])
    //远程依赖声明 标准依赖库
    implementation 'com.android.support:appcompat-v7:27.0.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
}
```



# 2 简单的活动创建

【主要内容】简单的活动创建。



## 活动

活动（Activity）是一种可以包含用户界面组件，主要用于和用户进行交互。一个应用程序可以包含零个或多个活动。

### 首先创建一个空活动

首先创建一个空活动 FirstActivity 在包的目录下

<a id="FirstActivity">FirstActivity.java</a>

```java
public class FirstActivity  extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //setContentView(R.layout.first_layout);
    }
}
```

### 创建和加载布局

然后创建和加载布局，在 `app/src/main/res` 目录下新建目录 layout 。然后对着layout目录右键 Layout resource file，新建布局文件 first_layout.xml，并添加一个按钮

first_layout.xml

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button_1"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
		android:text="Button 1"
        />
</LinearLaout>
```

`android:id` 给当前元素指定唯一标识符，之后可以在代码中对这个元素进行操作。 `@id/xxx` 是引用存在的资源， `@+id/xxx` 是指定义一个id。`layout_width` 和 `layout_height` 分别指宽度和高度的设置，`match_parent` 和 `warp_content` 分别指跟随父类和包含内容大小。

然后我们要在活动中加载这个布局，回到上面的[FirstActivity](#FirstActivity)。将注释的部分去掉注释，这里使用了 `setContentView` 方法传入first_layout布局文件的id  `R.layout.first_layout` 创建页面。其中项目中添加的任何资源都会放在 R 文件中生成一个相应的资源 id。

### 注册活动

在 AndroidManifest.xml 文件中注册活动，才能使活动生效。

AndroidManifest.xml

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.activitytest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        
        <!-- 注册FirstActivity活动 -->
        <!-- name字段表示活动名 
			 .FirstActivity 为 com.example.activitytest.FirstActivity 的缩写
			 由于 manifest 中通过 package 属性指定了程序包名 注册时可以省略
		-->
        <activity android:name=".FirstActivity"
        	android:label="This is FirstActivity">
            <!-- label 表示标题栏 主活的标题栏还会成为启动器中的程序名 -->
            <intent-filter>
                <!-- 将FirstActivity定义为主活动 -->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

这样FirstActivity就成为我们这个程序的主活动了，点击左面应用程序图标时会首先出现这个活动。另外，如果一个应用程序中没有声明任何一个活动为主活动，这个程序仍然是可以安装的，只是无法在启动器中看到或打开这个程序。这种程序一般都是作为第三方服务供其它应用在内部进行调用的。

现在我们主要构建的文件主要如下

```
ActivityTest ┬ ...
			 ├ app/src/main ┬ java/com... - FirstActivity.java
			 |              ├ res/layout - first_layout.xml
			 |              ├ AndroidManifest.xml
			 |              └ ...
             └ ...
```



## Toast (一种提醒方式 短小的信息)

Toast 是 Android 系统提供的一种短小消息的提醒方式，这些提醒消息不会占用任何屏幕空间，且在一段时间后自动消失。下面是Toast的使用方法。

首先需要定义一个弹出 Toast 的触发点，正好界面上有一个按钮，我们就让点击这个按钮的时候弹出一个Toast吧。在 `onCreate()` 方法中添加如下内容。

FirstActivity.java

```java
public class FirstActivity  extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.first_layout);
        // findViewById 用于通过id获取布局文件中的元素 返回一个View对象
        Button button1 = (Button) findViewById(R.id.button_1); //向下转型成 Button
        // 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) { // 重写 onClick() 方法
                // makeText 的三个参数
                // FirstActivity.this 第一个参数是 Context 这里填活动本身就可以
                // "..." 字符串 要显示的内容
                // Toast.LENGTH_SHORT Toast显示时长有两个内置常量 Toast.LENGTH_LONG 和 ~
                Toast.makeText(FirstActivity.this, "You clicked Button 1", 
                               Toast.LENGTH_SHORT).show(); // show()用于显示 Toast
            }
        });
        
    }
}
```

现在从新运行程序，点击按钮就会弹出提示，并很快消失。



## Menu 菜单的使用



如果活动中有大量菜单需要显示，这些菜单可能占用大量屏幕空间，Android 提供一种方式，可以让菜单得到展示的同时，还能不占用任何屏幕空间。

首先在 res 目录下新建一个 menu 文件夹，接着在文件夹下新建一个名叫 main.xml 的菜单文件。在文件中添加如下代码。

res/menu/main.xml

```xml
<!-- 菜单布局文件 -->
<menu xmlnx:android="http://schemas.android.com/apk/res/android">
	<!-- 创建两个菜单项 -->   
    <item android:id="@+id/add_item"
          android:title="Add"/>
    <item android:id="@+id/remove_item"
          android:title="Remove"/>  
</menu>
```

现在我们主要构建的文件主要如下

```
ActivityTest ┬ ...
			 ├ app/src/main ┬ java/com... - FirstActivity.java
			 |              ├ res ┬ layout - first_layout.xml
			 |              |     ├ menu - main.xml
			 |              |     └ ...
			 |              ├ AndroidManifest.xml
			 |              └ ...
             └ ...
```

回到 FirstActivity.java 中重写 `onCreateOptionsMenu()` 方法。

FirstActivity.java

```java
public class FirstActivity  extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.first_layout);
        // findViewById 用于通过id获取布局文件中的元素 返回一个View对象
        Button button1 = (Button) findViewById(R.id.button_1); //向下转型成 Button
        
        // 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) { // 重写 onClick() 方法
                // makeText 的三个参数
                // FirstActivity.this 第一个参数是 Context 这里填活动本身就可以
                // "..." 字符串 要显示的内容
                // Toast.LENGTH_SHORT Toast显示时长有两个内置常量 Toast.LENGTH_LONG 和 ~
                Toast.makeText(FirstActivity.this, "You clicked Button 1", 
                               Toast.LENGTH_SHORT).show(); // show()用于显示 Toast
            }
        });
        
        // 重写 onCreateOptionsMenu 方法
        @Override
        public boolean onCreateOptionsMenu(Menu menu) {
            // getMenuInflater() 能够得到 MenuInflater对象
            // 再调用它的 inflate() 方法就可以给当前活动创建菜单了
            // 第一个参数用于指定资源文件 第二个参数用于指定我们的菜单传入那个Menu对象中
            getMenuInflater().inflate(R.menu.main, menu);
            retutn true;
        }
        
        // 定义菜单的响应事件
        public boolean onOptionsItemSelected(MenuItem item) {
            switch (item.getItemId()) { // 通过item.getItemId()判断点击的哪一个菜单项
                case R.id.add_item:
                    Toast.makeText(this, "You clicked Add", 
                                   Toast.LENGT_SHORT).show();
                    break;
                case R.id.remove_item:
                    Toast.makeText(this, "You clicked Remove", 
                                   Toast.LENGT_SHORT).show();
                    break;
                default:
            }
            return true;
        }
        
    }
}
```

运行程序后可以看到在页面的标题栏的右侧有一个菜单按钮，点击后会出现菜单项，只会占用很小的一部分空间，点击响应的菜单项会出现对应的提示。



## 销毁活动 finish()

最简单的销毁活动的方法就是点击 Back 键。如果希望通过代码在销毁活动，Activity类提供了一个 `finish()` 方法。修改 FirstActivity.java 中按钮监听其中的代码，如下。

FirstActivity.java > button1.setOnClickListener()

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法button1.setOnClickListener(new View.OnClickListener() {    @Override    public void onClick(View v) { // 重写 onClick() 方法        finish(); //销毁当前活动    }});
```

重新运行程序，这时再点击按钮当前活动就被成功销毁了。



## 使用 Intent 在活动之间穿梭



### 前期准备

我们需要在com.example.activity包中再创建一个活动，命名为 SecondActivity，系统会自动生成 SecondActivity.java 和 second_layout.xml这两个文件。目录结构如下：

```
ActivityTest ┬ ...
			 ├ app/src/main ┬ java/com... ┬ FirstActivity.java
			 |              |             └ SecondActivity.java
			 |              ├ res ┬ layout ┬ first_layout.xml
			 |              |     |        └ second_layout.xml		 
			 |              |     ├ menu - main.xml
			 |              |     └ ...
			 |              ├ AndroidManifest.xml
			 |              └ ...
             └ ...
```

编辑 second_layout.xml 文件， res/layout/second_layout.xml

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button_2"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
		android:text="Button 2"
        />
</LinearLaout>
```

然后 编辑 SecondActivity.java

```java
public class SecondActivity  extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second_layout);
    }
}
```

最后在 AndroidManifest.xml 中注册，AndroidManifest.xml

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.activitytest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        
        <activity android:name=".FirstActivity"
        	android:label="This is FirstActivity">
            <intent-filter>
                <!-- 将FirstActivity定义为主活动 -->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    <!-- 注册SecondActivity活动 -->
	<activity android:name=".SecondActivity"></activity>
</manifest>

```

Intent 是 Android 程序中各组件之间进行交互的一种重要方式，==它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。== Intent 一般被用在启动活动、启动服务以及发送广播等场景。



### 显式 Intent

Intent 有多个构造函数的重载，其中一个就是 `Intent(Context packageContext, Class<?> cls)` 。这个构造函数的第一个参数 `Context ` 要求提供一个启动活动的上下文，第二个参数 `Class` 则指定想要启动的目标活动，通过这个构造函数就可以构建出 Intent 的“意图”。然后我们使用 Activity 类中的 `setActivity()` 方法，这个方法是专门由于启动活动的，它接收一个 Intent 参数，这里我们将构建好的 Intent 传入 `setActivity()` 方法就能启动目标活动了。

修改 FirstActivity.java 文件中的 按钮点击事件 FirstActivity.java > button1.setOnClickListener()

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        // 构建一个 Intent 表明意图 在 FirstActivity 活动的基础上打开 SecondActivity 活动
        Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
        // 启动目标活动
        startActivity(intent);
    }
});
```

重新执行程序，点击按钮，可以看到，能够成功启动 SecondActivity 这个活动。如果想要回到上一个活动，点击 Back 键就可以销毁当前活动，从而回到上一个活动了。



### 隐式 Intent

隐式 Intent 不会明确指出我们想要启动哪一个活动，而是指定一系列的更为抽象的 action 和 category 等信息，然后交由系统去分析这个 Intent 并帮助我们找出合适的活动去启动。

通过在 `<activity>` 标签下配置 `<intent-filter>` 的内容，可以指定当前活动能响应的 action 和 category，打开 AndroidManifest.xml，添加如下代码。

AndroidManifest.xml

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.activitytest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        
        <activity android:name=".FirstActivity"
        	android:label="This is FirstActivity">
            <intent-filter>
                <!-- 将FirstActivity定义为主活动 -->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    <!-- ------------------------------添加代码-------------------------------- -->
    <!-- 注册SecondActivity活动 -->
	<activity android:name=".SecondActivity">
    	<intent-filter>
            <!-- action 指明当前活动可以响应 
				 com.example.activitytest.ACTION_START 这个action 
			-->
            <action android:name="com.example.activitytest.ACTION_START" />
            
            <!-- category 提供附加信息 指明当前活动能够响应的 Intent 中可能带有的 category -->
            <!-- android.intent.category.DEFAULT 是一种默认的 category
 				 在调用 startActivity 方法时会自动将这个 category 添加到 intent 中
			-->
            <category android:name="android.intent.category.DEFAULT" />
		</intent-filter>
    </activity>
    <!-- ------------------------------添加代码-------------------------------- -->
</manifest>


```

这段代码说明只有 `<activity>` 和 `<category>` 中的内容同时能够匹配上 Intent中指定的action和category时，这个互动才能响应该 Intent。接着修改  FirstActivity.java 文件中的 按钮点击事件

 FirstActivity.java > button1.setOnClickListener

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        // 构建一个 Intent 表明 要启动 com.example.activitytest.ACTION_START 这个 action
        Intent intent = new Intent("com.example.activitytest.ACTION_START");
        // 启动目标活动
        startActivity(intent);
    }
});
```

这就是隐式 Intent 的结构，当然如果 category 中不是 默认的选项，就应该在 Intent 中添加响应的 category，基本形式如下：

AndroidManifest.xml > SecondActivity

```xml
<!-- 注册SecondActivity活动 -->
<activity android:name=".SecondActivity">
    <intent-filter>
        <!-- action 指明当前活动可以响应 
             com.example.activitytest.ACTION_START 这个action 
        -->
        <action android:name="com.example.activitytest.ACTION_START" />

        <!-- category 提供附加信息 指明当前活动能够响应的 Intent 中可能带有的 category -->
        <!-- category -> com.example.activitytest.MY_CATEGORY -->
        <category android:name="com.example.activitytest.MY_CATEGORY" />
    </intent-filter>
</activity>
```

 FirstActivity.java > button1.setOnClickListener

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        
        // 构建一个 Intent 表明 要启动 com.example.activitytest.ACTION_START 这个 action
        Intent intent = new Intent("com.example.activitytest.ACTION_START");
        // 向 intent 中添加 category "com.example.activitytest.MY_CATEGORY"
        initent.addCategory("com.example.activitytest.MY_CATEGORY");
            
        // 启动目标活动
        startActivity(intent);
    }
});
```

* 隐式 Intent 的其它用法

**调用系统浏览器** 

在活动的事件响应代码中可以这样写，例  FirstActivity.java > button1.setOnClickListener

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        
        // Intent.ACTION_VIEW 是一个 Android 系统内置的动作，
        // 常量值为 android.intent.action.VIEW
        Intent intent = new Intent(Intent.ACTION_VIEW);
       
        // Uri.parse() 将一个网址字符串解析成一个 Uir 对象
        // setData() 接收一个 Uir 对象
        initent.setData(Uri.parse("https://www.baidu.com"));
            
        // 启动浏览器加载指定网站
        startActivity(intent);
    }
});
```

点击 按钮就会转跳到百度的首页了。

与此对应，我们还可以在 `<intent-filter>` 标签中再配置一个 `<data>` 标签用于更精确地指定当前活动能够响应什么类型的数据。 `<data>` 标签中主要可以配置以下内容。

> - android:scheme		用于指定数据的协议部分，如 https 部分。
> - android:host              用于指定数据的主机部分，如 www.baidu.com 部分。
> - android:port              用于指定数据的端口部分，一般紧随在主机名之后。
> - android:path             用于指定主机名和端口之后的部分，如一段网址中跟在域名之后的内容。
> - android:mimeType  用于指定可以处理的数据类型，允许使用通配符的方式进行指定。

只有 `<data>` 标签中指定的内容和 Intent 中携带的 Data 完全一致时，当前活动才能够响应该 Intent。一般 `<data>` 标签中通常只会指定 https ，就会响应所有的 https 协议的 Intent 了。

除了 https 协议外，我们还可以指定很多其他协议，比如 geo 表示显示地理位置、tel 表示拨打电话。

例 拨打电话

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        
        // Intent.ACTION_DIAL 是一个 Android 系统内置的动作，
        Intent intent = new Intent(Intent.ACTION_DIAL);
       
        // Uri.parse() 将一个网址字符串解析成一个 Uir 对象
        // setData() 接收一个 Uir 对象
        initent.setData(Uri.parse("tel:10086"));
            
        // 启动浏览器加载指定网站
        startActivity(intent);
    }
});
```



### 向下一个活动传递数据

在启动活动时传递数据的思路很简单，Intent 中提供了一系列 `putExtra()` 方法可的重载，可以把数据暂存在 Intent 中，启动另一个活动后，只需要把这些数据再从 Intent中取出来就可以了。例：FirstActivity 中有一个字符串，现在想把这个字符串传递到 SecondActivity 中，就可以这样编写。

  FirstActivity.java > button1.setOnClickListener

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
  		String data = "Hello SecondActivity";
        Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
        // extra_data 是键值 data 是数据
        intent.putExtra("extra_data", data);
        // 启动目标活动
        startActivity(intent);
    }
});
```

然后在 SecondActivity.java 中调用 `getStringExtra()` 的方法传入键值就可以得到传递的字符串了

```java
public class SecondActivity  extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second_layout);
        
        // 获取用于启动 SecondActivity 的 Intent
        Intent intent = getIntent();
        // 提取字符串
        String data = intent.getStringExtra("extra_data");
        Log.d("SecondActivity", data);
    }
}
```

还有其他类型的方法  `getIntExtra()` 是传递整型数字的，  `getBooleanExtra()`  是传递布尔型的。

现在重新运行程序，在 FirstActivity 中点击按钮会转跳到 SecondActivity，查看 logcat 打印信息，会看到 data中的字符串已经被打印出来了。



### 返回数据给上一个活动

没有用于启动活动 Intent 来传递数据。但是 Activity 中还有一个 `startActivityForResult()` 方法也是用于传递数据。这个方法期望在活动销毁的时候能够返回一个结果给上一个活动。修改 FirstActivity.java 中的代码。

  FirstActivity.java > button1.setOnClickListener

```java
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        
        Intent intent = new Intent(FirstActivity.this, SecondActivity.class);		
        // 第一个参数是 Intent 第二个参数是请求码 用于在之后的回调中判断数据l
        startActivityForResult(intent, 1);
    }
});
```

这里使用 `startActivityForResult()` 方法来启动 SecondActivity ，请求码只要是一个唯一值就可以。记下来在 SecondActivity 中给按钮注册点击事件，并在点击事件中添加返回数据的逻辑，代码如下。

SecondActivity.java

```java
public class SecondActivity  extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second_layout);
        Button button2 = (Button) findViewById(R.id.button_2);
        button2.setOnClickListener(new View.OnAClickListener() {
            @Override
            public void onClick(View v){
                // 构建 Intent 仅用于传递参数
                Intent intent = new Intent();
                intent.putExtra("data_return", "Hello FirstActivity");
                // setResult 接收两个参数
                // 第一个参数 用于上一活动返回处理结果，RESULT_OK/RESULT_CANCELED
                // 第二个参数则把带有数据的 Intent 传递回去
                setResult(RESULT_OK, intent);
                // 销毁活动
                finish();
            }
        });
        
    }
}

```

然后重写 `onActivityResult()`

  FirstActivity.java > onActivityResult

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	// requestCode 为启动活动时的请求码 该案例为1 用于判断数据源
    switch (requestCode) {
        case 1:
            if (resultCode == RESULT_OK) { // resultCode 用于判断结果是否处理成功
                String returnedData = data.getStringExtra("data_return");
                Log.d("FirstActivity", returnedData); //打印
            }
            break;
    	default;
    }   
}
```

重新运行程序，在 FirstActivity 界面打开 SecondActivity ，然后在SecondActivity 界面点击 Button 2 按钮会回到 FirstActivity，这时查看 logcat 的打印信息。

如果用户不在 SecondActivity 中并不是通过点击按钮，而是通过按下 Back 键回到 FirstActivity，可以通过在 SecondActivity 中重写 `onBackPressed()` 方法来解决这个问题。这样按下 Back 键就会执行 `onBackPressed()` 中的逻辑了。 

SecondActivity.java > onBackPressed

```java
@Override
public void onBackPressed() {
    Intent intent = new Intent();
    intent.putExtra("data_return", "Hello FirstActivity");
    setResult(RESULT_OK, intent);
    finish();
}
```



# 3 活动生命周期

【主要内容】活动生命周期。

## 活动生命周期



### 返回栈

Android 每启动一个新活动，就会覆盖在原活动之上，然后点击Back就会销毁最上面的活动，下面的一个活动就会重新显示出来。Android 是使用任务（Task）来管理活动的，一个任务就是一组存放在站立的活动的集合，这个栈也被称作返回栈（Back Stack）。栈是一种后劲先出的数据结构，在默认情况下，每当启动了一个新的活动时，它就会在返回栈中入栈，并处于栈顶位置，每当销毁一个活动时，处于栈顶的活动就会出栈，这时前一个入栈的活动就会重新处于栈顶的位置，系统总会显示处于栈顶的活动给用户。



**活动的状态**

* 1 运行态：当一个活动位于栈顶时，这时活动就处于运行状态。系统最不愿意回收的就是处于运行状态的活动，因为这会带来非常差的用户体验。
* 2 暂停态：当一个活动不再处于栈顶位置，但仍然可见时，活动就进入了暂停状态。处于暂停状态的活动仍然是完全存活着的，系统也不愿意去回收这种活动（因为它还是可见的，回收可见的东西都会在用户体验方面有不好的影响），只有在内存极低的情况下，系统才会去考虑回收这种活动。
* 3 停止状态：当一个活动不再处于栈顶位置，并且完全不可见的时候就进入了停止状态。系统仍然会为这种活动保存响应的状态和成员变量，但是不是完全可靠的，当其他地方需要内存时，处于停止状态的活动有可能会被系统回收。
* 4 销毁状态：当一个活动从返回栈中移除后就变成了销毁状态。系统最倾向于回收这种状态的活动，从而保证手机的内存充足。

**活动的生存期**

Activity 类中定义了7个回调方法，覆盖了活动生命周期的每一个环节。

* `onCreate()` 每个活动都会重写这个方法，它会在活动第一次被创建的时候调用。完成初始化、加载布局、绑定事件等。
* `onStart()` 这个方法在活动由==不可见==变为==可见==的时候调用。
* `onResume()` 这个方法在活动准备好和用户进行交互的时候调用。此时的==活动一定位于返回栈顶，且处于运行状态==。
* `onPause()` 这个方法在==系统准备去启动或者恢复另一个活动的时候调用==。在这个方法中通常将一些消耗CPU的资源释放掉，以及保存一些关键数据，但是这个方法的执行速度一定要快，不然会影响到新的栈顶活动的使用。
* `onStop()` 这个方法在活动==完全不可见==的时候调用。他和onPause的主要区别在于，如果启动的新活动是一个对话框式的活动，那么 onPause 方法会得到执行，而 onStop方法并不会执行。
* `onDestroy()` 这个方法在==活动被销毁之前调用==，之后活动的状态将变为销毁状态。

* `onRestart()` 这个方法在==活动由停止状态变为运行状态之前调用==，也就是活动被重新启动了。

以上7中方法中除了 `onRestart()` 方法，其它都是两两相对的，从而分为3种生存周期。

* **完整生存周期** 活动在 `onCreate()` 和 `onDestroy()` 之间就是完整的生存周期。
* **可见生存期** 活动在 `onStart()` 和 `onStop()` 之间就是可见生存周期。在可见僧从期内，活动对用户总是可见的，即便有可能无法与用户交互。可以通过对 `onStart()` 对资源进行加载和 `onStop()` 对资源进行释放，从而保证处于停止状态的活动不会占用过多内存。
* **前台生存期** 活动在 `onPause()` 和 `onRestart()` 之间是前台生存期。在前台生存期内，活动总是处于运行状态，此时活动是可以和用户进行交互的。



## 活动以外被回收

当活动进入到了停止状态，是有可能被系统回收的，如果上一个被停止的活动中存有临时数据和状态，这时从当前活动返回上一个活动时，上一个活动就会被重新创建一次。为了解决这个问题 Activity 中提供了一个 `onSaveIntanceState()` 回调方法，这个方法可以保证在活动被回收之前一定会被调用，这个方法中会携带一个 Bundle 类型的参数，Bundle 提供一系列方法用于保存数据。

例如；在主活动中重写 `onSaveIntanceState()` 方法：

```java
@Override
protected void onSaveIntanceState(Bundle outState) {
    super.onSaveIntanceState(outState);
    String tempData = "Something you just typed";
    outState.putString("data_key", tempData);
}
```

继续修改主活动的 `onCreate()` 方法

```java
protected void onCreate(Bundle saveIntanceState) {
    super.oncreate(saveIntanceState);
    Log.d(TAG, "onCreate");
    setContentView(R.layout.activity_main);
    if(saveIntanceState != null) {
        String tempData = saveIntanceState.getString("data_key");
        Log.d("data_key", tempData);
    }
    // ...
}
```



## 活动的启动模式



启动模式一共分有4种，分别是 standard、singleTop、singleTask 和 singleInstance，可以在 AndroidManifest.xml 中通过给 `<activity>` 标签指定 `android:launchMode` 属性来选择启动模式。

### standard

standard 是默认的启动模式，在不进行显示指定的情况下，所有的活动都会自动使用这种启动模式。在standard 模式（即默认情况）下，每当启动一个新活动，它就会在返回栈中入栈，并处于栈顶的位置。

在 ActivityTest 项目的基础上做一些修改，ActivityTest 项目目录结构如下

```
ActivityTest ┬ ...
			 ├ app/src/main ┬ java/com... ┬ FirstActivity.java
			 |              |             └ SecondActivity.java
			 |              ├ res ┬ layout ┬ first_layout.xml
			 |              |     |        └ second_layout.xml		 
			 |              |     ├ menu - main.xml
			 |              |     └ ...
			 |              ├ AndroidManifest.xml
			 |              └ ...
             └ ...
```

修改 FirstActivity.java 中的 `onCreate()` 方法，FirstActivity.java > onCreate()

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 打印 FirstActivity+实例信息
    Log.d("FirstActivity", this.toString());
    setContentView(R.layout.first_layout);
    Button button1 = (Button) findViewById(R.id.button_1);
    button1.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // 启动一个新的 FirstActivity 实例
            Intent intent = new Intent(FirstActivity.this, FirstActivity.class);
            startActivity(intent);
        }
    });       
}
```

重新运行程序点击两次按钮，可以看到调试信息中打印了三条信息，分别是三个不同的 FirstActivity 实例，此时返回栈中会有三个 FirstActivity 实例，需要连续三次 Back 键才能退出程序。



### singleTop



singleTop 模式：如果当前活动指定为 singleTop 模式，在活动启动时如果发现返回栈顶已经是该活动，则认为可以直接使用它，不会再创建新的活动实例。具体方法，修改 AndroidManifest.xml 中 FirstActivity 的启动模式。

AndroidManifest.xml > FirstActivity 

```xml
<activity android:name=".FirstActivity"
          android:launchMode="singleTop"
          android:label="This is FirstActivity">
    <!-- 指定为 singleTop 模式 -->
    <intent-filter>
        <!-- 将FirstActivity定义为主活动 -->
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

重新运行程序，无论点击多少次按钮都不会有新的信息打印出来，也就是不会有新的实例被创建。但是如果当 FirstActivity 不在栈顶位置时，再启动 FirstActivity 还是会创建新的实例。



### singleTask

singleTask 模式用于解决在某个活动的整个应用程序的上下文中只存在一个实例。singleTask 模式的活动在每次启动时系统首先会再返回栈中检查是否纯在给活动的实例，如果发现已经存在则直接使用该实例，并且把这个活动之上的所有活动统统出栈，如果没有就会创建一个新的活动实例。具体方法，首先修改 AndroidManifest.xml 中 FirstActivity 的启动模式。

AndroidManifest.xml > FirstActivity 

```xml
<activity android:name=".FirstActivity"
          android:launchMode="singleTask"
          android:label="This is FirstActivity">
    <!-- 指定为 singleTask 模式 -->
    <intent-filter>
        <!-- 将FirstActivity定义为主活动 -->
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

然后在 FirstActivity.java 中修改 `onCreate()` 方法并添加 `onRestart()` 方法，并打印日志。FirstActivity.java > onCreate() & onRestart()

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 打印 FirstActivity+实例信息
    Log.d("FirstActivity", this.toString());
    setContentView(R.layout.first_layout);
    Button button1 = (Button) findViewById(R.id.button_1);
    button1.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // 启动一个新的 SecondActivity 实例
            Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
            startActivity(intent);
        }
    });       
}

@Override
protected void onRestart() {
    super.onRestart();
    Log.d("FirstActivity", "onRestart");
}
```

然后在 SecondActivity.java 中修改 `onCreate()` 方法并添加 `onDestroy()` 方法，并打印日志。SecondActivity.java > onCreate() & onDestroy()

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 打印 SecondActivity+实例信息
    Log.d("SecondActivity", this.toString());
    setContentView(R.layout.second_layout);
    Button button2 = (Button) findViewById(R.id.button_2);
    button2.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // 启动一个新的 FirstActivity 实例
            Intent intent = new Intent(SecondActivity.this, FirstActivity.class);
            startActivity(intent);
        }
    });       
}

@Override
protected void onDestroy() {
    super.onDestroy();
    Log.d("SecondActivity", "onDestroy");
}
```

重新运行程序，在 FirstActivity 界面点击按钮进入到 SecondActivity，然后在 SecondActivity 界面点击按钮（这里不会新建 FirstActivity 实例而是会先调用原来FirstActivity的onRestart，然后再销毁SecondActivity），又会重新进入到 FirstActivity。



### singleInstance

指定为 singleInstance 模式的活动会启用一个新的返回栈来管理这个活动（其实如果 singleTask 模式指定了不同的taskAffinity，也会启动一个新的返回栈）。用于解决不用应用程序共享活动实例的问题。具体方法，首先修改 AndroidManifest.xml 中 SecondActivity的启动模式。

AndroidManifest.xml > SecondActivity

```xml
<!-- 注册SecondActivity活动 -->
<activity android:name=".SecondActivity"
          android:launchMode="singleInstance">
    <!-- 指定为 singleInstance 模式 -->
    <intent-filter>    
        <action android:name="com.example.activitytest.ACTION_START" />
        <category android:name="com.example.activitytest.MY_CATEGORY" />
    </intent-filter>
</activity>
```

然后再修改 FirstActivity 中的 `onCreate()` 方法，FirstActivity.java > onCreate() 

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 打印 FirstActivity+实例信息
    Log.d("FirstActivity", "Task id is " + getTaskId());
    setContentView(R.layout.first_layout);
    Button button1 = (Button) findViewById(R.id.button_1);
    button1.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // 启动一个新的 SecondActivity 实例
            Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
            startActivity(intent);
        }
    });       
}
```

然后再修改 SecondActivity中的 `onCreate()` 方法，SecondActivity.java > onCreate()

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 打印 SecondActivity+实例信息
    Log.d("SecondActivity", "Task id is " + getTaskId());
    setContentView(R.layout.second_layout);
    Button button2 = (Button) findViewById(R.id.button_2);
    button2.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // 启动一个新的 SecondActivity 实例
            Intent intent = new Intent(SecondActivity.this, ThirdActivity.class);
            startActivity(intent);
        }
    });       
}
```

添加第三个活动ThirdActivity，修改 ThirdActivity.java > onCreate()

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 打印 ThirdActivity+实例信息
    Log.d("ThirdActivity", "Task id is " + getTaskId());
    setContentView(R.layout.third_layout);
}
```

现在重新运行程序，在 FirstActivity 界面点击按钮进入 SecondActivity，在 SecondActivity 界面点击按钮进入 ThirdActivity 。观察打印信息，SecondActivity 的Task id不同于 FirstActivity 和 ThirdActivity ，然后按下 Back 键，程序从 ThirdActivity 返回到了 FirstActivity ，再次点击 Back 键，回到 SecondActivity ，最后点击 Back 退出程序。解释，Back 只会按栈的结构处理活动，只有当前的栈没有活动实例了才会跳到下一个栈。



## 活动其它功能



### 知晓当前是哪一个活动

在 ActivityTest 项目的基础上修改，需要在目录 com.example.activity 下新建一个 BaseActivity 类，建立后的ActivityTest 项目目录结构如下

```
ActivityTest ┬ ...
			 ├ app/src/main ┬ java/com... ┬ FirstActivity.java
			 |              |             ├ SecondActivity.java
			 |              |             ├ ThirdActivity.java
			 |              |             └ BaseActivity.java
			 |              ├ res ┬ layout ┬ first_layout.xml
			 |              |     |        ├ second_layout.xml
			 |              |     |        └ third_layout.xml
			 |              |     ├ menu - main.xml
			 |              |     └ ...
			 |              ├ AndroidManifest.xml
			 |              └ ...
             └ ...
```

BaseActivity 为普通的java类 继承自 AppCompatActivity，并重写 `onCreate()` 方法

BaseActivity.java

```java
public class BaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("BaseActivity", getClass().getSimpleName());
    }
}
```

接下来让 BaseActivity 成为所有活动的父类。修改各个活动的继承结构，重新运行程序并分别进入三个活动的界面，观察打印信息能看到在不影响继承特性的情况下实现了打印活动类名的功能。



### 随时随地退出程序

如何实现不用层层 Back 实现退出程序？需要用一个专门的集合类对所有的活动进行管理就可以了。新建一个 ActivityCollector 类作为活动管理器，代码如下：

ActivityCollector.java

```java
public class ActivityCollector {
    // 活动列表
    public static List<Activity> activities = new ArrayList<>();
    // 添加活动
    public static void addActivity(Activity activity) {
        activities.add(activity);
    }
    // 移除活动
    public static void removeActivity(Activity activity) {
        activities.remove(activity);
    }
    // 销毁所有活动
    public static void finishAll() {
        for (Activity activity : activities) {
            if (!activity.isFinishing()) {
                activity.finish();
            }
        }
    }
}
```

在 BaseActivity.java 中加入活动管理方法，BaseActivity.java

```java
public class BaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("BaseActivity", getClass().getSimpleName());
        ActivityCollector.addActivity(this); // 添加活动管理
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        ActivityCollector.removeActivity(this); // 移除活动管理
    }
}
```

以后不管在什么地方，只要调用 `ActivityCollector.finishAll()` 方法就能直接退出程序。当然还可以在销毁所有活动的代码后边再加上杀掉当前进程的代码，以保证程序完全退出，杀掉进程的代码如下所示：

```java
android.os.Process.killProcess(android.os.Process.myPid());
```



### 启动活动的最佳写法

启动活动的方法首先通过 Intent 构建出当前的 “意图”，然后在调用 `startActivity()` 或 `startActivityForResult()` 方法将活动启动起来，如果有数据要从一个活动传递到另一个活动也可以借助 Intent 来完成。但是这样活动传递的数据各不相同错综复杂，对维护也很不利，所以考虑将整个启动过程封装成一个方法，例如在 SecondActivity 中添加一个 `actionStart()` 方法。

SecondActivity.java

```java
public class SecondActivity extends BaseActivity {
    // ...
	public static void actionStart(Context context, String data1, String data2) {
        Intent intent = new Intent(context, SecondActivity.class);
        intent.putExtra("param1", data1);
        intent.putExtra("param2", data2);
        context.startActivity(intent);
    }
    // ...
}
```

现在在其它活动中仅需要一行代码就可以启动 SecondActivity 了，如下

```java
button1.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        // 启动 SecondActivity
        SecondActivity.actionStart(FirstActivity.this, "data1", "data2");
    }
});
```



# 4 简单和常用的控件简介

【主要内容】简单和常用的控件简介。



## 常用控件的使用方法

### TextView

例：

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <TextView
    	android:id="@+id/text_view"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
		android:text="This is TextView" />
    
</LinearLaout>
```

这是最简单的 TextView，显示默认位置为左上角，我们可以修改对齐方式，现在加入字段 `android:gravity` 来指定文字对齐方式，可选值有 `top` `bottom` `left` `right` `center` 等，可以用 `|` 来同时指定多个值，这里用 `center` 设置，效果等同于 `center_vertical | center_horizontal` 表示文字在垂直水平方向都居中。

另外还可以修改文字的大小和颜色进行修改，`android:textSize` 可以指定文字的大小，通过 `android:textColor` 指定文字的颜色，在 Android 中字体大小使用 sp 作为单位。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <TextView
    	android:id="@+id/text_view"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
        android:gravity="center"
        android:textSize="24sp"
        android:textColor="#00ff00"
		android:text="This is TextView" />
    
</LinearLaout>
```



### Button

Button 的属性和 TextView 是差不多的，布局文件里面设置的文字是 “Button”，如果不加任何设置，最终的显示结果会是 “BUTTON”，这是系统会默认对 Button 中所有的英文字母自动进行大写转换，可以使用 `android:textAllCapps` 来禁用这一默认属性。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
		android:text="Button"
        android:textAllCapps="false" />
    
</LinearLaout>
```

接下来可以在活动中为 Button 的点击事件注册一个监听器

```java
// 匿名注册
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstnceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View V) {
                // 逻辑
            }
        });
    }
}
```

下面是用接口的方式实现注册

```java
// 接口方式注册
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstnceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findById(R.id.button);
        button.setOnClickListener(this);
    }
    
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                // 逻辑
                break;
            default:
                break;
        }
    }
}
```



### EditText

EditText 允许用户在控件里输入和编辑内容，并可以在程序中对这些内容进行处理。使用 `android:hint` 指定在输入框内的提示内容。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <EditText 
    	android:id="@+id/edit_text"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
        android:hint="Type something here" />
    
</LinearLaout>
```

随着输入的内容不断增多， EditText 会被不断地拉长。由于高度属性是 `wrap_content` 因此它总能包含住里面的内容，可以使用 `android:maxLines` 属性来解决这个问题。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <EditText 
    	android:id="@+id/edit_text"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
        android:hint="Type something here" 
        android:maxLines="2" />
    
</LinearLaout>
```



```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstnceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findById(R.id.button);
        editText = (EditText) findViewById(R.id.edit_text);
        button.setOnClickListener(this);
    }
    
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                String inputText = editText.getText().toString();
                Toast.makeText(MainActivity.this, inputText, 
                              Toast.LENGTH_SHORT).show();
                break;
            default:
                break;
        }
    }
}
```





### ImageView

ImageView 是用于在界面展示图片的一个控件，图片通常放在以 “drawable” 开头的目录下的。目前我们的项目中有一个空的 drawable 目录，这里我们在 res 目录下新建一个 drawable-xhdpi 目录，然后将准备好的两张图片 img_1.png 和 img_2.png 复制到该目录当中。



```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <ImageView 
    	android:id="@+id/image_view"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
        android:src="@drawable/img_1" 
         />
    
</LinearLaout>
```





```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstnceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findById(R.id.button);
        editText = (EditText) findViewById(R.id.edit_text);
        imageView = (ImageView) findViewById(R.id.image_view);
        button.setOnClickListener(this);
    }
    
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                imageView.setImageResource(R.drawable.img_2);
                break;
            default:
                break;
        }
    }
}
```



### ProgressBar

ProgressBar 用于在界面上显示一个进度条，表示我们在加载一些数据。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <ProgressBar  
    	android:id="@+id/progress_bar "
        android:layout_width="match_parent"
        android:layout_height="warp_content" 
         />
    
</LinearLaout>
```

重新运行，屏幕中会后一个圆形进度条正在旋转，如果需要在数据加载完成后让进度条消失就需要用Android控件的可见属性。可以通过 `android:visibility` 进行指定，可选值有3种： `visible` `invisible` `gone` 。`visible` 表示控件是可见的，这个是默认值，不指定可见属性是，控件都是可见的。`invisible` 表示控件是不可见的，但是仍然占据原来的大小和位置可以理解为控件变成透明状态了。 `gone` 则表示控件不仅是不可见的而且不占屏幕的任何空间。可以用代码设置控件的可见属性，使用 `setVisiblity()` 方法，可以传入 `View.VSIBLE` `View.INVSIBLE` `View.GONE` 来控制。

下面尝试实现，点击一下按钮让进度条消失，再点击一下按钮让进度条出现的效果。修改MainActivity 中的代码，如下所示：

MainActivity.java

```java
public class MainActivity extends AppCompatActivity 
    					  implements View.OnClickListenter {
    private EditText editText;
    private ImageView imageView;
    private ProgressBar progressBar;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstnceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findById(R.id.button);
        editText = (EditText) findViewById(R.id.edit_text);
        imageView = (ImageView) findViewById(R.id.image_view);
        progressBar = (ProgressBar)findById(R.id.progress_bar);
        button.setOnClickListener(this);
    }
    
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                if (progressBar.getVisibility() == View.GONE) {
					progressBar.setVisibility(View.VISIBLE);
                } else {
                    progressBar.setVisibility(View.GONE);
                }
                break;
            default:
                break;
        }
    }
}
```

另外，我们还可以给 ProgressBar 指定不同的样式，刚刚是圆形进度条，通过style属性可以将它指定成水平进度条。指定成水平进度条后，可以通过 `android:max` 属性给进度条设置一个最大值，然后在代码中动态地修改进度条进度。修改 activity_main.xml 中的代码：

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <ProgressBar  
    	android:id="@+id/progress_bar "
        android:layout_width="match_parent"
        android:layout_height="warp_content"
        style="?android:attr/progressBarStyleHorizontal"
        android:max="100"
        />
    
</LinearLaout>
```

```java
public class MainActivity extends AppCompatActivity 
    					  implements View.OnClickListenter {
    // ...
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                int progress = progressBar.getProgress();
                progress = progress + 10;
                progressBar.setProgress(progress);
                break;
            default:
                break;
        }
    }
}
```

再次运行程序，每点击一次按钮，就会获取进度条当前进度，然后加上10作为更新后的进度。



### AlertDialog

AlertDialog 可以在当前界面弹出一个对话框，这个对话框是置顶于所有界面元素之上的，能够屏蔽掉其它控件的交互能力，因此 AlertDialog 一般用于提示一些非常重要的内容或者警告信息。

MainActivity.java

```java
public class MainActivity extends AppCompatActivity 
    					  implements View.OnClickListenter {
    // ...
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                AlertDialog.Builder dialog = new AlertDialog.
                    								Builder (MainActivity.this);
                dialog.setTitle("This is Dialog");
                dialog.setMessage("Something important.");
                dialog.setCancelable(false);
                dialog.setPositiveButton("OK", new DialogInterface.
                	OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                          
                        }
                    });
				dialog.setNegativeButton("Cancel", new DialogInterface.
                	OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which){
                            
                        }
                    });
                dialog.show();
                break;
            default:
                break;
        }
    }
}
```

首先通过 AlertDialog.Builder 创建一个 AlertDialog 的实例，然后可以为这个对话框设置标题、内容、可否取消等属性，并设置按钮点击监听器，和点击事件处理，接下来调用 `setPositiveButton()` 方法为对话框设置确定按钮的点击事件，调用 `setNegativeButton()` 方法为取消按钮的点击事件，最后调用 `show()` 方法将对话框显示出来。



### ProgressDialog

与 AlertDialog 相似 ProgressDialog 都是弹出一个对话框，能够屏蔽掉其它控件的交互能力。不同的是，ProgressDialog 会在对话框中显示一个进度条，一般用于表示当前操作比较耗时，让用户耐心等待。

```java
public class MainActivity extends AppCompatActivity 
    					  implements View.OnClickListenter {
    // ...
    @Override
    protected void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                ProgressDialog progressDialog = new ProgressDialog
                	(MainActivity.this);
                progressDialog.setTitle("This is ProgressDialog");
                progressDialog.setMessage("Loading...");
                progressDialog.setCancelable(true);
                progressDialog.show();
                break;
            default:
                break;
        }
    }
}
```

注意：如果在 `setCancelable()` 中传入了 false ，表示该控件是不能通过 Back 键取消掉的，当数据加载完成后必须调用对应的 `.dismiss()` 方法来关闭对话框。



## 4 种基本布局



### 线性布局

#### android:orientation

LinearLayout 又称作线性布局，这个布局会将它所包含的控件在线性方向上依次排列。可以通过 `android:orientation` 属性指定排列方向 vertical 是垂直方向，horizontal 是水平方向。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button1"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
		android:text="Button 1"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button2"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
		android:text="Button 2"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button3"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
		android:text="Button 3"
        android:textAllCapps="false" />
    
</LinearLaout>
```

在 Layout 中添加了 3 个 Button ，每个 Button 的长和宽都是 `warp_content` ，并指定了排列方向是 `vertical` 。如果需要横向排列的话只需要修改下面这一段

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
```

#### android:layout_gravity

用于指定控件布局的对齐方式，可选值与 `android:gravity` 差不多，但是当 LinearLayout 是 horizontal 排列的时候，只有垂直方向上的对齐方式才会生效，因为此时水平方向上的长度是不固定的，每添加一个控件，水平方向上的长度都会发生改变，因而无法指定该方向上的对齐方式。同样 vertical 只有在水平方向上的对齐方式才会生效。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button1"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_gravity="top"    
		android:text="Button 1"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button2"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_gravity="center_vertical"
		android:text="Button 2"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button3"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_gravity="bottom"
		android:text="Button 3"
        android:textAllCapps="false" />
    
</LinearLaout>
```



#### android:layout_width

android:layout_width 这个属性允许我们使用比例的方式来指定控件的大小，例如下方的一个消息发送界面，包含一个文本框和一个发送按钮。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">

    <TextView
    	android:id="@+id/input_message"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1"      
		android:text="This is TextView" />
    
    <Button
    	android:id="@+id/send"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1"
		android:text="Send"
        android:textAllCapps="false" />
   
</LinearLaout>
```

这里把两个控件的宽度都指定成了 `0dp` ，然后在将 `android:layout_weight` 属性指定为 1 ，表示两个控件平分屏幕宽度，当然如果分别设置为 3 和 2 那么这两个控件将分别占据屏幕宽度的 2/5 和 3/5。如果文本框指定了 `android:layout_weight` 属性为 1，将按钮的 `android:layout_width` 指定为 warp_content ，则 按钮大小为包含内容，文本框将占据剩余所有宽度。



### 相对布局



RelativeLayout 又称作相对布局，相对布局可以通过相对定位的方式让控件出现在布局的任何位置。

```xml
<RelativeLayout xmlnx:android="http://schemas.android.com/apk/res/android"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button1"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
		android:text="Button 1"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button2"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"
		android:text="Button 2"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button3"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_cenerInParent="true"
		android:text="Button 3"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button4"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentBottom="true"
		android:text="Button 4"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button5"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentBottom="true"
		android:text="Button 5"
        android:textAllCapps="false" />    
    
</LinearLaout>
```

这样，五个按钮将分别位于屏幕的左上、右上、中间、左下、右下的位置上。分别是与父布局左上角、右上角、居中、左下角、右下角对齐。另外，控件还能相对于其它控件进行定位。

```xml
<RelativeLayout xmlnx:android="http://schemas.android.com/apk/res/android"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">

    <Button
    	android:id="@+id/button3"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_cenerInParent="true"
		android:text="Button 3"
        android:textAllCapps="false" />	
    <Button
    	android:id="@+id/button1"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_above="@id/button3"
        android:layout_toLeftOf="@id/button3"
		android:text="Button 1"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button2"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_above="@id/button3"
        android:layout_toRightOf="@id/button3"
		android:text="Button 2"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button4"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_below="@id/button3"
        android:layout_toLeftOf="@id/button3"
		android:text="Button 4"
        android:textAllCapps="false" />
    <Button
    	android:id="@+id/button5"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_below="@id/button3"
        android:layout_toRightOf="@id/button3"
		android:text="Button 5"
        android:textAllCapps="false" />    
    
</LinearLaout>
```

这个布局效果是五个控件紧挨在一起并且为左上、右上、中间、左下、右下的分布。

除了这些还有 `android:layout_alignLeft` `android:layout_alignRight` `android:layout_alignTop` `android:layout_alignBottom` 表示一个控件的各种边缘和另一个控件对应的边缘对齐。



### 帧布局

FrameLayout 又称作帧布局，这种布局会将所有控件都会默认摆放在布局的左上角。默认的情况下控件会按照添加的先后顺序叠加在一起。可以通过使用 `layout_gravity` 属性来指定空间布局中的对齐方式。

```xml
<FrameLayout xmlns:android+"http://schemas.android.com/apk/res/android"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <TextView
    	android:id="@+id/text_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:text="This is TextView"
        />
    <ImageView
        android:layout_width="warp_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:src="@mipmap/ic_launcher" 
        />       
            
</FrameLayout>
```

由于 FrameLayout 的定位方式欠缺，导致应用场景也比较少。



### 百分比布局

在百分比布局中，直接指定控件在布局中所占的百分比，由于 LinearLayout 本身已经支持按比例指定控件大小了，，因此百分比布局只为 FrameLayout 和 RelativeLayout 进行功能扩展，提供了 PercentFrameLayout 和 PercentRelativeLayout 这两个全新布局。

使用百分比布局首先需要在项目的 `build.gradle` 中添加百分比布局库的依赖。

app/build.gradle

```groovy
dependencies {
	compile fileTree(dir: 'libs', include:['*.jar'])
    compile 'com.android.support:appcompat-v7:24.2.1'
    compile 'com.android.support:percent:24.2.1'
    testCompile 'junit:junit:4.12'
}
```

修改布局中的代码

```xml
<android.support.percent.PercentFrameLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
	
    <Button
    	andorid:id="@+id/button1"
        android:text="Button 1"
        android:layout_gravity="left|top"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />
    <Button
    	andorid:id="@+id/button2"
        android:text="Button 2"
        android:layout_gravity="right|top"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />
    <Button
    	andorid:id="@+id/button3"
        android:text="Button 3"
        android:layout_gravity="left|bottom"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />
    <Button
    	andorid:id="@+id/button4"
        android:text="Button 4"
        android:layout_gravity="right|bottom"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />    
</android.support.percent.PercentFrameLayout>
```





## 自定义控件

所有的布局都是直接或间接继承 ViewGroup 的。

### 引入布局

两个按钮和一个文本显示框就能构成一个标题栏，如果每一个布局都要写一遍就会导致代码大量重复。这时可以用引用布局来解决这个问题，新建一个布局 title.xml

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
    android:layout_width="macth_parent"
    android:layout_height="wrap_content"
    android:background="@drawable/title_bg">

    <Button
    	android:id="@+id/title_back"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/title_bg"
		android:text="Back"
        android:textColor="#fff" />
    
    <TextView
    	android:id="@+id/title_text"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1"
        android:layout_gravity="center"
		android:text="This is TextView" 
        android:textColor="#fff" 
        android:textSize="24sp"/>
    
    <Button
    	android:id="@+id/title_edit"
        android:layout_width="warp_content"
        android:layout_height="warp_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/edit_bg"
		android:text="Edit"
        android:textColor="#fff" />
   
</LinearLaout>
```

下面将在 activity_main.xml 中使用这个布局。

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
    		 android:layout_width="macth_parent"
             android:layout_height="wrap_content">
		
    <include layout="@layout/title" />

</LinearLaout>
```

然后隐藏自带的标题栏

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activit_main);
        ActionBar actionbar = getSupportActionBar();
        if (actionbar != null) {
            actionbar.hide();
        }
    }
}
```

### 创建自定义控件

引入布局解决了编写重复代码的问题，但是如果标题栏中的返回按钮，不管在那一个活动中，这个按钮的功能都是相同的，即销毁当前活动。而如果每个活动都需要新注册一遍返回按钮的点击事件，无疑会增加很多重复的代码，这种情况最好是使用自定义控件

新建 TitleLayout 继承自 LinearLayout，让它成为自定义的标题栏控件

```java
public class TitleLayout extends LinearLayout {
    
    public TitleLayout(Context contet, AttributeSet attrs) {
        super(context, attrs);
        LayoutInflater.from(context).inflate(R.layout.title, this);
    }
}
```

首先我们重写了 LinearLayout 中带有两个参数的构造函数，在布局中引入 TitleLayout 控件就会调用这个构造函数。然后在构造函数中需要对标题栏布局进行动态加载，需要借助 LayoutInflater 来实现了。通过 LayoutInflater 的 `from()` 方法可以构建出一个 LayoutInflater 对象，然后调用 `inflate()` 方法就可以动态加载一个布局文件， `inflate()` 方法接收两个参数，第一个参数是要加载布局文件的 id，第二个是给加载好的布局再添加一个父布局，这里我们想要指定为 TitleLayout，所以直接传入 this。

自定义控件已经创建好了，然后需要在布局文件中添加这个自定义控件，修改 activity_main.xml 中的代码，如下：

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">

    <packagename.TitleLayout
        android:layout_width="macth_parent"
        android:layout_height="warp_content"
        />
   
</LinearLaout>
```

添加自定义控件和添加普通控件的方式基本一样，只不过在添加自定义控件的时候，我们需要指明控件的完整类名，包名在这里是不可省略的。

下面尝试为标题中的按钮注册点击事件，修改 TitleLayout 中的代码，如下所示：

```java
public class TitleLayout extends LinearLayout {
    
    public TitleLayout(Context contet, AttributeSet attrs) {
        super(context, attrs);
        LayoutInflater.from(context).inflate(R.layout.title, this);
        // 按钮注册点击事件
        Button titleBack = (Button) findViewById(R.id.title_back);
        Button titleEdit = (Button) findviewById(R.id.title_edit);
        titleBack.setOnClickListener(new onClickListener() {
        	@Override
            public void onClick(View v) {
                ((Activity) getContext()).finish();
            }
        });
        titleEdit.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(getContext(), "You clicked Edit button",
                              Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

这样就自动实现好了点击事件。



## ListView



ListView 允许用户通过上下滑动的方式将屏幕外的数据滚动得到屏幕内。

### ListView的简单用法

首先新建一个 ListView 项目，并让 Android Studio 自动帮我们创建好活动。然后修改 activity_main.xml 中的代码，如下：

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

接下来修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {

    private String[] data = {"Apple", "Banana", "Orange", "Watermelon",
            "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango",
            "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango"};
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ArrayAdapter<String> adapter = new ArrayAdapter<String>(
                MainActivity.this, android.R.layout.simple_list_item_1, data
        );
        ListView listView = (ListView) findViewById(R.id.list_view);
        listView.setAdapter(adapter);
    }
}
```

这里我们需要使用适配器将数组中的数据传递给 ListView ，其中比较好的是 ArrayAdapter。它可以通过泛型来指定要适配的数据类型，然后在构造函数中要把适配的数据传入。这里使用的构造类型需要三个参数，第一个是当前上下文，第二个为 ListView 子布局的id，以及要适配的数据，这里使用 `simple_list_item_1` 这是一个内置的布局文件，里边只有一个 TextView， 可用于简单地显示一段文本。

最后调用 `listView.setAdapter(adapter)` 将构建好的适配器对象传递进去，运行程序，就能看见期待的效果了。

### 定制 ListView 界面

除了显示文字，还能对 ListView 的界面进行定制，让她可以显示更加丰富的内容，准备好一组图片没别对应上面的文字，接下来在每一个文字的旁边都会添加一个图样。

定义一个实体类，作为 ListView 适配器的适配类型。新建 Picture.java

```java
package com.example.listview;

public class Picture {
    private String name; // 资源名字
    private int imageId; // 对应图片的 id

    public Picture(String name, int imageId) {
        this.name = name;
        this.imageId = imageId;
    }

    public String getName() {
        return name;
    }

    public int getImageId() {
        return imageId;
    }
}
```

然后需要为 ListView 的子项指定一个自定义的布局，在 layout 目录下新建 picture_item.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/picture"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/picture_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="50dp"/>

</LinearLayout>
```

接下来需要自定义一个适配器，这个适配器继承自 ArrayAdapter，并将泛型指定为 Picture类。新建类 PictureAdapter.java

```java
public class PictureAdapter extends ArrayAdapter<Picture> {
    private int resourceId;

    public PictureAdapter(Context context, int textViewResourceId,
                          List<Picture> object) {
        super(context, textViewResourceId, object);
        resourceId = textViewResourceId;
    }

    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, 
                        @NonNull ViewGroup parent) {
        Picture picture = getItem(position); //获取当前 Picture 实例
        /*
         * 这里第三个参数为 false 表示只让我们在父布局中声明的layout属性生效，
         * 但不为这个 View 添加父布局，因为一旦 view 有了父布局之后，就不能再
         * 添加到 ListView 中了。
         */
        View view = LayoutInflater.from(getContext()).inflate(resourceId, 
                                                              parent, false);
        ImageView pictureImage = (ImageView) view.findViewById(R.id.picture);
        TextView pictureName = (TextView) view.findViewById(R.id.picture_name);
        pictureImage.setImageResource(picture.getImageId()); // 设置图片
        pictureName.setText(picture.getName()); //设置文字
        return view;
    }
}

```

PictureAdapter 重写了父类的一组构造函数，用于将上下文、ListView 子布局的id和数据都传过来。另外又重写了 `getView()` 方法，这个方法在每个子项被滚动到屏幕内部的时候都会调用。在 `getView()` 方法中，首先通过 `getItem()` 得到当前项的实例，然后在用 `LayoutInflater()` 来为这个子项传入布局。

接着在 MainActivity.java 中调用

```java
public class MainActivity extends AppCompatActivity {

    private List<Picture> pictureList = new ArrayList<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initPictures();
        PictureAdapter adapter = new PictureAdapter(
                MainActivity.this, R.layout.picture_item, pictureList
        );
        ListView listView = (ListView) findViewById(R.id.list_view);
        listView.setAdapter(adapter);
    }

    private void initPictures() {
        for (int i =0; i < 2; i++) {
            Picture apple = new Picture("Apple", R.drawable.a);
            pictureList.add(apple);
            Picture banana = new Picture("Banana", R.drawable.b);
            pictureList.add(banana);
            Picture orange = new Picture("Orange", R.drawable.c);
            pictureList.add(orange);
            Picture watermelon = new Picture("Watermelon", R.drawable.d);
            pictureList.add(watermelon);
            Picture pear = new Picture("Pear", R.drawable.e);
            pictureList.add(pear);
            Picture grape = new Picture("Grape", R.drawable.f);
            pictureList.add(grape);
            Picture pineapple = new Picture("Pineapple", R.drawable.g);
            pictureList.add(pineapple);
            Picture strawberry = new Picture("Strawberry", R.drawable.h);
            pictureList.add(strawberry);

        }
    }
}
```

运行即可达到效果。



### 提升 ListView 的运行效率



ListView 的运行效率很低，因为在 PictureAdapter 的 `getView()` 方法中，每次都将布局重新加载了一遍，当 ListView 快速滚动的时候，这就会成为性能的瓶颈。

在 `getView()` 方法中还有一个 convertView 参数，这个参数用于将之前加载好的布局进行缓存，以便之后可以进行重新调用。修改 PictureAdapter.java 中的代码

```java
public class PictureAdapter extends ArrayAdapter<Picture> {
    private int resourceId;

    public PictureAdapter(Context context, int textViewResourceId,
                          List<Picture> object) {
        super(context, textViewResourceId, object);
        resourceId = textViewResourceId;
    }

    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, 
                        @NonNull ViewGroup parent) {
        Picture picture = getItem(position); //获取当前 Picture 实例
        
        View view;
        if (convertView == null){
            view = LayoutInflater.from(getContext()).inflate(
                resourceId, parent, false);    
        } else {
            view = convertView;
        }
        
        ImageView pictureImage = 
            (ImageView) view.findViewById(R.id.picture);
        TextView pictureName = 
            (TextView) view.findViewById(R.id.picture_name);
        pictureImage.setImageResource(picture.getImageId());
        pictureName.setText(picture.getName());
        return view;
    }
}

```

这里使用了if对缓存状态进行判断，如果为空就去加载布局，否则就直接对缓存重用，这样大大提高了 ListView 的效率。缺点：还会调用 `view.findViewById()` 方法来获取一次控件的实例。

这里可以借助一个 ViewHolder 来对这部分性能进行优化，修改 PictureAdapter 如下

```java
public class PictureAdapter extends ArrayAdapter<Picture> {
    private int resourceId;

    public PictureAdapter(Context context, int textViewResourceId,
                          List<Picture> object) {
        super(context, textViewResourceId, object);
        resourceId = textViewResourceId;
    }

    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, 
                        @NonNull ViewGroup parent) {
        Picture picture = getItem(position); //获取当前 Picture 实例

        View view;
        ViewHolder viewHolder;
        if (convertView == null){
            view = LayoutInflater.from(getContext()).inflate(
                resourceId, parent, false);
            viewHolder = new ViewHolder();
            viewHolder.pictureImage = (ImageView) view.findViewById(R.id.picture);
            viewHolder.pictureName = (TextView) view.findViewById(R.id.picture_name);
            view.setTag(viewHolder); // 将 ViewHolder 存储在 View 中
        } else {
            view = convertView;
            viewHolder = (ViewHolder) view.getTag(); // 重新获取 ViewHolder
        }

        viewHolder.pictureImage.setImageResource(picture.getImageId());
        viewHolder.pictureName.setText(picture.getName());
        return view;
    }
    public class ViewHolder {
        public ImageView pictureImage;
        public TextView pictureName;
	}
}
```

经过两次优化，ListView 的运行效率已经非常不错了。



### ListView 的点击事件

下面在 MainActivity.java 中添加点击事件的相应，

```java
public class MainActivity extends AppCompatActivity {

     List<Picture> pictureList = new ArrayList<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initPictures();
        PictureAdapter adapter = new PictureAdapter(
                MainActivity.this, R.layout.picture_item, pictureList
        );
        ListView listView = (ListView) findViewById(R.id.list_view);
        listView.setAdapter(adapter);
		// 点击响应
        listView.setOnItemClickListener(
        	new AdapterView.OnItemClickListener() {
            	@Override
            	public void onItemClick(AdapterView<?> parent, 
            						View view, int position, long id) {

                	Picture picture = pictureList.get(position);
                	Toast.makeText(MainActivity.this, picture.getName(), 
                				   Toast.LENGTH_SHORT).show();
            }
        });       
    }
    private void initPictures() {
        for (int i =0; i < 2; i++) {
            Picture apple = new Picture("Apple", R.drawable.a);
            pictureList.add(apple);
            Picture banana = new Picture("Banana", R.drawable.b);
            pictureList.add(banana);
            Picture orange = new Picture("Orange", R.drawable.j);
            pictureList.add(orange);
            Picture watermelon = new Picture("Watermelon", R.drawable.d);
            pictureList.add(watermelon);
            Picture pear = new Picture("Pear", R.drawable.e);
            pictureList.add(pear);
            Picture grape = new Picture("Grape", R.drawable.f);
            pictureList.add(grape);
            Picture pineapple = new Picture("Pineapple", R.drawable.g);
            pictureList.add(pineapple);
            Picture strawberry = new Picture("Strawberry", R.drawable.h);
            pictureList.add(strawberry);

        }
    }
}
```

运行程序，点击每个表项，就能看到弹出对应名字的 Toast  提示框。

## RecyclerView 的基本用法



和百分比布局类似， RecycleView 也属于新的控件，使用 RecycleView 需要在项目的 build.gradle 中添加相应的依赖库才行，打开 app/build.gradle 文件，在dependencies 闭包中添加如下内容：

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:24.2.1'
    compile 'com.android.support:recyclerview-v7:24.2.1'
    testCompile 'junit:junit:4.12'
}
```

添加结束后点击 Sync Now同步。修改 activity_main.xml 中的代码，如下所示：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
	
    <android.support.v7.widget.RecyclerView
             android:id="@+id/recycler_view"
             android:layout_width="match_parent"
             android:layout_height="match_parent" />
    
</LinearLayout>
```

这里想要使用 RecyclerView 来实现和 ListView 相同的效果，为了方便，我们直接从 ListView 项目中把图片复制过去就可以了顺便将 Picture.java 和 picture_item.xml 也复制过来。

接下来需要为 RecyclerView 准备一个适配器，新建 PictureAdapter 类，让这个适配器继承自 RecyclerView.Adapter，并将泛型指定为 PictureAdapter.ViewHolder。其中 ViewHolder 是我们在 PictureAdapter 中定义的内部类，代码如下

PictureAdapter.java

```java
public class PictureAdapter extends 
    RecyclerView.Adapter<PictureAdapter.ViewHolder> {
    
    private List<Picture> mPictureList;
    
    /* 首先定义一个内部类 ViewHolder 继承自 PictureAdapter.ViewHolder
     * 然后 ViewHolder 的构造函数中要传入一个 View 参数 这个参数通常就是
     * RecyclerView 子项的最外层布局，那么就可以通过 findViewById() 方
     * 法来获取到布局中的 ImageView 和 TextView 的实例了。
     */
    static class ViewHolder extends PictureAdapter.ViewHolder {
        ImageView pictureImage;
        TextView pictureName;
        
        public ViewHolder(View view) {
            super(view);
            
            ImageView pictureImage = 
                (ImageView) view.findViewById(R.id.picture);
            
        	TextView pictureName = 
                (TextView) view.findViewById(R.id.picture_name);
            
        	pictureImage.setImageResource(picture.getImageId());
        }
    }
    
    /* PictureAdapter 中也有一个构造函数，这个方法用于把要展示的数据源传进来
     * 并赋值给一个全局变量 mPictureList ，然后我们后续的操作都将在这个数据
     * 的基础上进行。
     */
    public PictureAdapter(List<Picture> pictureList) {
        mPictureList = pictureList;
    }
    
    // 由于 ViewHolder 继承自 PictureAdapter.ViewHolder
    // 那么就必须重写 onCreateViewHolder onBindViewHolder getItemCount
    /* onCreateViewHolder 用于创建一个 ViewHolder 实例
     * 并将 picture_item 布局加载进来，然后创建一个 ViewHolder 实例
     * 并把加载好的布局传入到构造函数中 最后返回 ViewHolder 的实例
     */
    @Override 
    public ViewHolder onCreateViewHolder(ViewGroup parent, int position) {
        View view = LayoutInflater.from(parent.getContext())
            .inflate(R.layout.fruit_item, parent, false);
        ViewHolder holder = new ViewHolder(view);
        return holder;
    }
    
    /* onBindViewHolder 用于对 RecycleView 子项的数据进行赋值
     * 在每个子项被滚动到屏幕内的时候执行，通过 position 参数得到
     * 当前项的 Picture 然后再将数据设置到 ViewHolder 的
     * ImageView 和 TextView 中去
     */
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        Picture picture = mPictureList.get(position);
        holder.pictureImageResource(picture.getImageId());
        holder.pictureName.setText(picture.getName());
    }
    
    /* 返回数据源的长度 即 RecyclerView 一共有多少子项 */
    @Override
    public int getItemCount() {
        return mPictureList.size();
    } 
}
```

适配器准备好之后，就可以开始使用 RecyclerView 了，修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {

     List<Picture> pictureList = new ArrayList<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initPictures();
        // 获取 RecyclerView 的实例
        RecyclerView recyclerView = 
            (RecyclerView) findViewById(R.id.recycler_view);
        // 创建一个 LinearLayoutManager 对象
        // 这里指的是使用的 LinearLayoutManager 线性布局的意思
        // 实现与 ListView 类似的效果
        LinearLayoutManager layoutManager = 
            new LinearLayoutManager(this);
        // 将 LinearLayoutManager 设置到 recyclerView 中
        recyclerView.setLayoutManager(layoutManager);
        // 创建 PictureAdapter 实例 并将数据传入
        PictureAdapter adapter = new PictureAdapter(pictureList);
        // 调用 setAdapter() 完成适配器设置
        recyclerView.setAdapter(adapter);
    }
    private void initPictures() {
        for (int i =0; i < 2; i++) {
            Picture apple = new Picture("Apple", R.drawable.a);
            pictureList.add(apple);
            Picture banana = new Picture("Banana", R.drawable.b);
            pictureList.add(banana);
            Picture orange = new Picture("Orange", R.drawable.j);
            pictureList.add(orange);
            Picture watermelon = new Picture("Watermelon", R.drawable.d);
            pictureList.add(watermelon);
            Picture pear = new Picture("Pear", R.drawable.e);
            pictureList.add(pear);
            Picture grape = new Picture("Grape", R.drawable.f);
            pictureList.add(grape);
            Picture pineapple = new Picture("Pineapple", R.drawable.g);
            pictureList.add(pineapple);
            Picture strawberry = new Picture("Strawberry", R.drawable.h);
            pictureList.add(strawberry);

        }
    }
}
```

### 实现横向滚动和瀑布流布局

ListView 扩展性不好，只能实现纵向滚动的效果，如果想进行横向滚动就需要 RecycleView 来实现。

首先对 picture_item.xml 修改，如果要实现横向滚动的话，应该把 picture_item.xml 里的元素改成垂直排列才比较合理，改动如下

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="100dp"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/picture"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal" />

    <TextView
        android:id="@+id/picture_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"/>

</LinearLayout>
```

这里将 LinearLayout 改成垂直方向排列，并把宽度设为 100dp。如果用 wrap_content 的话，RecyclerView 的子项就会有长有短，非常不美观；如果 match_parent 的话会导致宽度过长，一个子项占满整个屏幕。并将 ImageView 和 TextView 都设置成了在布局中水平居中，并使用 `android:layout_marginTop="10dp"` 属性让文字和图片之间保持一些距离。

接下来修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {

     List<Picture> pictureList = new ArrayList<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initPictures();

        RecyclerView recyclerView = 
            (RecyclerView) findViewById(R.id.recycler_view);

        LinearLayoutManager layoutManager = 
            new LinearLayoutManager(this);
        // 设置布局排列 默认是纵向 ，LinearLayoutManager.HORIZONTAL 为横向
        layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);

        recyclerView.setLayoutManager(layoutManager);

        PictureAdapter adapter = new PictureAdapter(pictureList);

        recyclerView.setAdapter(adapter);
    }
    // ...
}
```

除了横向排布还有瀑布式的排列布局，瀑布式的布局需要使用 `GridLayoutManager` 和 `StaggeredGridLayoutManager` 两种内置的布局排列布局， `GridLayoutManager` 可以用于网格布局， `StaggeredGridLayoutManager` 可以实现瀑布式布局，这里不再详细介绍。



## 编写界面的最佳实践



### 制作 Nine-Patch 图片

略

### 编写精美的聊天界面



# 5 碎片（Fragment）

【主要内容】碎片（Fragment）

碎片是一种可以嵌入在活动中的UI片段，可以粗略地将碎片理解成一个“迷你活动”。碎片是为了更加充分利用平板电脑的大屏幕而生的，如果把手机界面等比例放大，放在平板上这显然是不合理的，不仅使用起来不方便，而且浪费了大量的屏幕空间。如果使用两个碎片将屏幕按不同比例分割开，分别完成不同的功能，就能很好地解决这个问题。

## 碎片的使用方式

新建一个 FragmentTest 项目。

### 碎片的简单用法

这里先在一个活动中添加两个碎片，并让这两个碎片平分活动空间。

新建一个左侧布局 left_fragment.xml,

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
	<Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:text="Button" 
            />
</LinearLayout>
```

新建一个右侧布局 right_fragment.xml，并将这个布局背景设置成绿色，

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:background="#00ff00"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
	<TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_gravity="center_horizontal"
              android:textSize="20sp"
              android:text="This is right fragment" 
              />
</LinearLayout>
```

接着新建一个 LeftFragment 类，并让它继承自 Fragment。注意，这里可能会有两个不同包下的 Fragment 供你选择，一个是内置的 `android.app.Fragment` ，另一个是 support-v4 库中的 `android.support.v4.app.Fragment` 。这里强烈建议使用 support-v4 库中的 Fragment。

下面编写一下 LeftFragment.java 

```java
public class LeftFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                            Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.left_fragment, container, false);
        return view;
    }
}
```

这里通过重写 Fragment 的 onCreateView() 方法，然后在这个方法中通过 LayoutInflater 的 `inflate()` 方法将刚刚定义的 left_fragment 布局动态加载进来，RightFragment.java 也一样。

```java
public class RightFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                            Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.right_fragment, container, false);
        return view;
    }
}
```

接下来修改 activity_main.xml

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
    
    <Fragment
    	android:id="@+id/left_fragment"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1" />

        <Fragment
    	android:id="@+id/right_fragment"
        android:name="com.example.fragmenttest.RightFragment"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1" />
    
</LinearLaout>
```

运行程序，可以看到平板屏幕两边个出现一个界面，左边含有一个按钮，右边的背景为绿色。



### 动态添加碎片

动态添加碎片可以在程序运行的时候动态地添加到活动中。

在上一部分的代码基础上继续完善，新建 another_right_fragment.xml，代码如下

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:background="#ffff00"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
	<TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_gravity="center_horizontal"
              android:textSize="20sp"
              android:text="This is another right fragment" 
              />
</LinearLayout>
```

新建 AnotherRightFragment.java 作为另一个右侧碎片

```java
public class AnotherRightFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                            Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.right_fragment, container, false);
        return view;
    }
}
```

下面将展示如何将它动态地添加到活动中，修改 activity_main.xml

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
    
    <Fragment
    	android:id="@+id/left_fragment"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1" />

    <FrameLayout
    	android:id="@+id/right_fragment"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1">
    </FrameLayout> 
    
</LinearLaout>
```

`FrameLayout` 是一种简单布局（前面提到过），所有控件默认都会摆在布局的左上角。下面我们将在代码中向 FrameLayout 中添加内容，实现动态加载碎片。修改 MainActivity.java 中的代码。

```java
public class MainActivity extends AppCompatActivity 
    implements View.OnClickListener {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(this);
        replaceFragment(new RightFragment());
    }
    
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.button:
                // 1 创建待加载的碎片实例 new AnotherRightFragment()
                replaceFragment(new AnotherRightFragment());
                break;
            default:
                break;
        }
    }
    
    private void replaceFragment(Fragment fragment) {
        // 2 获取 FragmentManager， 在活动中可以直接通过调用 getSupportFragmentManager() 方法得到
        FragmentManager fragmentManager = getSupportFragmentManager();
        // 3 开启一个事务，通过调用 beginTransaction() 方法开启。
        FragmentTransaction transaction = 
            fragmentManager.beginTransaction();
        // 4 向容器内添加或替换碎片，一般使用 replace() 方法实现，需要传入容器的 id 和待添加的碎片实例。
        transaction.replace(R.id.right_layout, Fragment);
        // 5 提交事务，调用 commit() 方法来完成。
        transaction.commit();
    }
    
}
```



* 总结

> 动态加载碎片主要分为一下5个步骤
>
> （1）创建待加载的碎片实例。
>
> （2）获取 FragmentManager， 在活动中可以直接通过调用 getSupportFragmentManager() 方法得到。
>
> （3）开启一个事务，通过调用 beginTransaction() 方法开启。
>
> （4）向容器内添加或替换碎片，一般使用 replace() 方法实现，需要传入容器的 id 和待添加的碎片实例。
>
> （5）提交事务，调用 commit() 方法来完成。



### 在碎片中模拟返回栈

在程序中添加碎片后，按下返回键不是退出碎片，而是程序会直接退出。如果想模拟类似于返回栈的效果，按下 Back 键可以回到上一个碎片，就需要 FragmentTransaction 的 addToBackStack() 方法，可以用于将一个事务添加到返回栈中，修改 MainActivity.java 中的代码。

```java
public class MainActivity extends AppCompatActivity 
    implements View.OnClickListener {
    
    // ...
    
    private void replaceFragment(Fragment fragment) {
        // 2 获取 FragmentManager， 在活动中可以直接通过调用 getSupportFragmentManager() 方法得到
        FragmentManager fragmentManager = getSupportFragmentManager();
        // 3 开启一个事务，通过调用 beginTransaction() 方法开启。
        FragmentTransaction transaction = 
            fragmentManager.beginTransaction();
        // 4 向容器内添加或替换碎片，一般使用 replace() 方法实现，需要传入容器的 id 和待添加的碎片实例。
        transaction.replace(R.id.right_layout, Fragment);
        //------------添加返回栈--------------
        transaction.addToBackStack(null);
        // 5 提交事务，调用 commit() 方法来完成。
        transaction.commit();
    }
    
}
```



### 碎片和活动之间进行通信

虽然碎片都是嵌入在活动中显示的，但是碎片和活动都是各自存在于一个独立的类当中的，他们之间并没有那么明显的方式来直接进行通信。如果想要在活动中调用碎片里的方法，FragmentManager 提供的 findFragmentById() 方法，可以在活动中得到相应碎片的实例，类似于 findViewById() 的方法。

掌握了如何在活动中调用碎片里的方法，那在碎片中又该怎样调用活动里的方法就可以调用 `getActivity()` 方法来得到和当前碎片相关联的活动实例，代码如下：

```
MainActivity activity = (MainActivity) getActivity();
```

有了活动实例之后，在碎片中调用活动里的方法就变得轻而易举了。另外当碎片中需要使用 Context 对象时，也可以使用 `getActivity()` 方法，因为获利到的活动本身就是一个 Context 对象。



## 碎片的生命周期



### 碎片的状态和回调



* 碎片的状态

> 1. 运行状态：当一个碎片是可见的，并且它所关联的活动正处于运行状态时，该碎片也处于运行状态。
> 2. 暂停状态：当一个活动进入暂停状态时（由于另一个未沾满屏幕的活动被添加到了栈顶），与它相关联的可见碎片就会进入到暂停状态。
> 3. 停止状态：当一个活动进入停止状态时，与它相关的碎片就会进入到停止状态，或者通过调用 FragmentTransaction 的 remove() , replace() 方法将碎片从活动中移除，但如果在事务提交之前调用 addToBackStack() 方法，这时的碎片也会进入到停止状态。总的来说进入停止状态的碎片对用户来说完全不可见的，有可能会被系统回收。
> 4. 销毁状态：碎片总是依附于活动存在的，因此当活动被销毁时，与它相关联的碎片就会进入到销毁状态。或者通过调用 FragmentTransaction 的 remove() , replace() 方法将碎片从活动中移除，但如果在事务提交之前==没有==调用 addToBackStack() 方法，这时的碎片也会进入到销毁状态。



同样地，Fragment 类中也提供了一系列的回调方法，下面介绍一部分：

* onAttach()。当碎片和活动建立关联的时候调用。
* onCreateView()。为碎片创建视图（加载布局）时调用。
* onActivityCreated()。确保与碎片相关联的活动一定已经创建完毕的时候调用。
* onDestroyView()。当与碎片关联的视图被移除的时候调用。
* onDetach().当碎片和活动解除关联的时候调用。



## 动态加载布局的技巧



### 使用限定符

如果你经常使用平板电脑，因为平板电脑的屏幕足够大，完全可以同时显示下两页的内容，但手机的屏幕一次就只能显示一页的内容，因此两个页面要分开显示。

如果需要程序自动判断应该使用单页模式还是双页模式就需要借助限定符（Qualifiers）来实现了。修改 FragmentTest 项目中的 activity_main.xml 文件，代码如下：

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
    
    <Fragment
    	android:id="@+id/left_fragment"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
  
</LinearLaout>
```

将多余的代码都删掉，只留一个左侧碎片，并让它充满整个父布局。接着在 res 目录下新建 layout-large 文件夹，在这个文件夹下新建一个布局，也叫 activity_main.xml，代码如下：

res/layout-large/activity_main.xml

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="horizontal"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
    
    <Fragment
    	android:id="@+id/left_fragment"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="1" />

    <FrameLayout
    	android:id="@+id/right_fragment"
        android:layout_width="0dp"
        android:layout_height="warp_content"
        android:layout_weight="3">
    </FrameLayout> 
    
</LinearLaout>
```

然后注释掉 MainActivity.java 中注释掉，并在平板和手机上运行程序，即可看到平板上为双页模式而手机上为单页模式。在工程目录中可以看到 /layout/activity_main.xml 布局只包含一个碎片，即单页模式，而 /layout-large/activity_main.xml 包含两个碎片，即双页模式。其中 large 就是一个限定符，那些屏幕被认为是 large 的就会自动加载 layout-large 文件夹下的布局，而小屏的设备还是会加载 layout 文件夹下的布局。

虽然 large 成功解决了单双页的判断问题，但是 large 到底有多大呢？所以有的时候为了更加灵活地为不同设备加载布局，就可以使用最小宽度限定符（Smallest-width Qualifiers）了。

最小宽度限定符允许我们对屏幕的宽度制定一个最小值（以dp为单位），然后以这个最小值为临界点，屏幕宽度大于这个值得设备就加载一个布局，屏幕小于这个宽度就加载另一个布局。

例如：在 res 目录下新建 layout-sw600dp 文件夹，然后在这个文件夹下新建 activity_main.xml 布局。这样就以为这屏幕宽度以 600dp 为边界加载不同的布局。



## 简易版的新闻应用

仅介绍项目组织结构

```
ActivityTest ┬ ...
	  ├ app/src/main ┬ java/com... ┬ MainActivity.java
	  |              |             ├ News.java
	  |              |             ├ NewsContentFragment.java
	  |              |             ├ NewsContentActivity.java	             
	  |              |             ├ NewsTitleFragment.java	             
	  |              |             └ NewsAdapter.java
	  |              ├ res ┬ layout ┬ activty_main.xml
	  |              |     |        ├ news_content_frag.xml 
	  |              |     |        ├ news_content.xml 
	  |              |     |        ├ news_title_frag.xml 
	  |              |     |        └ news_item.xml 	 
	  |              |     ├ layout-sw600dp - activty_main.xml
	  |              |     └ ...
	  |              ├ AndroidManifest.xml
	  |              ├ build.gradle			 
	  |              └ ...
      └ ...
```



* News.java：title 字段表示标题，content 字段表示新闻内容。
* news_content_frag.xml：新闻内容布局。
* NewsContentFragment.java：继承自 Fragment，用于分别获取控件，和加载布局（双页模式）。添加 `refresh()` 方法。
* NewsContentActivity.java：在`onCreate()` 中通过 Intent 获取到了传入的新闻的标题和内容然后调用 FragmentManager 的 findFragmentById 的方法得到 NewsContentFragment 实例，接着调用它的 refresh 方法并将新闻的标题和内容传入，就可以把这些数据显示出来，并重写 `actionStart()` 方法。
* news_content.xml：引用 NewsContentFragment 相当于自动加载 news_content_frag 布局。
* news_title_frag.xml ：用于显示新闻列表。
* news_item.xml：作为 RecycleView 子项的布局，只有一个 TextView 用于显示标题。
* NewsTitleFragment.java：展示新闻列表碎片，重写 `onActivityCreated()` 方法，通过在活动中是否能找到一个 id 为 news_content_layout 的 View 来判断当前是双页模式还是单页模式，所以需要设计让这个 id 为 news_content_layout 的 View 只在双页模式中出现。
* layout/activty_main.xml：单页模式布局，id 为 news_title_layout 并只加载一个新闻标题的碎片。
* layout-sw600dp/activty_main.xml：同时引入两个碎片，并将新闻内容的碎片放在了一个 FrameLayout 的布局下，并将这个布局的 id 设置为 news_content_layout 。
* NewsAdapter.java：RecycleView 的适配器，继承自 java：RecycleView.Adapter<NewsAdapter.ViewHolder> 在 NewsTitleFragment.java 中使用。用于加载新闻。如果是单页模式直接启动一个新活动，如果是双页模式刷新页面。
* MainActivity.java：主程序入口，加载主页面，初始化列表。



# 6 全局喇叭详解广播机制

【主要内容】全局喇叭详解广播机制：为了方便于进行系统级别的消息通知，Android 也引入了一套类似广播消息机制。



## 广播机制简介

Android 中的每个应用程序都可以对知己感兴趣的广播进行注册，这样该程序就只会接收到自己所关心的广播内容。Android 提供了一套完整的 API，允许应用程序自由地发送和接收广播。发送广播的方法广播的方法其实之前稍微提到过，在之前的 Intent 部分。而接收广播的方法需要引入一个新概念——广播接收器（Broadcast Receiver）。

Android 中的广播的类型主要可以分为一下两大类：

* 标准广播（Normal broadcast）：是一种完全异步执行的广播，在广播发出之后，说有的广播接收器几乎都会在同一时刻接收到这一条广播消息，因此它们之间没有任何先后顺序可言，也意味着它是无法被截断的。
* 有序广播（Ordered broadcast）：则是一种同步执行的广播，在广播发出之后，同一时刻只会有一个广播接收器能够收到这条广播消息，当这个广播接收器中的逻辑执行完毕后，广播才会继续传递。所以此时的广播接收器是有先后顺序的，优先级高的广播接收器就可以先接受到广播消息，并且前面的广播接收器还可以截断正在传递的广播，这样后边的广播接收器就无法收到广播消息了。



## 接收系统广播



### 动态注册监听网络变化

广播接收器可以自由地对自己感兴趣的广播进行注册，这样当有相应的广播发出时，广播接收器就能够收到该广播，并在内部处理响应的逻辑。注册广播方式一般有两种，在代码中注册（动态注册）和在 AndroidManifest.xml 中注册（静态注册）。

创建广播接收器：只需要新建一个类，让它继承自 BroadcastReceiver，并重写父类的 onReceive() 方法就行了。这时当有广播到来时，onReceive() 方法就会得到执行，具体的逻辑就可以在这个方法中处理。

* 动态注册示例：监听网络变化的程序，新建一个 BroadcastTest 项目，然后修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {
   
    private IntentFilter intentFilter;
    
    private NetworkChangeReceiver networkChangeReceiver;
    
    @Override 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);            
        setContentView(R.layout.hello_world_layout);
        // 创建一个 IntentFilter 实例
        intentFilter = new IntentFilter();
        // 添加一个 android.net.conn.CONNECTIVITY_CHANGE 的 action
        // 监听 android.net.conn.CONNECTIVITY_CHANGE 的广播
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver = new NetworkChangeReceiver();
        // 注册 就会会接收所有值为 android.net.conn.CONNECTIVITY_CHANGE 的广播
        registerReceiver(networkChangeReceiver, intentFilter);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 动态注册一定要注销
        unregisterReceiver(networkChangeReceiver);
    }
    
    class NetworkChangeReceiver extends BroadcastReceiver {
        
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "network changes", 
                           Toast.LENGTH_SHORT).show();
        }
    }
}
```

上述代码实现了网络状态变化的监听，现在试着改进

```java
public class MainActivity extends AppCompatActivity {
   
    private IntentFilter intentFilter;
    
    private NetworkChangeReceiver networkChangeReceiver;
    
    @Override 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);            
        setContentView(R.layout.hello_world_layout);
        // 创建一个 IntentFilter 实例
        intentFilter = new IntentFilter();
        // 添加一个 android.net.conn.CONNECTIVITY_CHANGE 的 action
        // 监听 android.net.conn.CONNECTIVITY_CHANGE 的广播
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver = new NetworkChangeReceiver();
        // 注册 就会会接收所有值为 android.net.conn.CONNECTIVITY_CHANGE 的广播
        registerReceiver(networkChangeReceiver, intentFilter);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 动态注册一定要注销
        unregisterReceiver(networkChangeReceiver);
    }
    
    class NetworkChangeReceiver extends BroadcastReceiver {
        
        @Override
        public void onReceive(Context context, Intent intent) {
            ConnectivityManager connectionManager = 
                (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo networkInfo = connectionManager.getActiveNetworkInfo();
            if (networkInfo != null && networkInfo.isAvailable()){
                Toast.makeText(context, "network is available", 
                           Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(context, "network is unavailable", 
                           Toast.LENGTH_SHORT).show();
            }

        }
    }
}
```

另外访问系统中的网络权限需要在 AndroidManifest.xml 中添加以下内容。

```xml
<uses-permission android:name="android.permission.ACCES_NETWORK_STATE" />
```



### 静态注册实现开机启动

动态注册广播接收器可以自由地控制注册与注销，在灵活性方面有很大的优势，但是它也存在着一个缺点，即必须要在程序启动之后才能接收到广播，因为注册的逻辑是写在 onCreate() 方法中的。

静态注册的方式能够用实现在程序未启动的情况下就能接收到广播。这里准备让程序收一条开机广播，当收到这条广播时就可以在 onReceive() 方法里执行相应的逻辑，从而实现开机自启。可以使用 Android Studio 提供的快捷方式来创建这个广播接收器，右击 com.pakagename -> New -> Other -> Broadcast Receiver，会弹出窗口，将广播接收器命名为 BootCompleteReceiver，Exported 属性表示是否允许这个广播接收器接收本程序以外的广播，Enable 属性表示是否启用这个广播接收器。勾选这两个属性，点击 Finish 完成创建。然后修改 BootCompleteReceiver 中的代码。

```java
public class BootCompleteReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        // 弹出消息框
        Toast.makeText(context, "Boot Complete", 
                       Toast.LENGTH_LONG).show();
    } 
}
```

另外，静态广播接收器一定要在 AndroidManifest.xml 中注册才能使用，如果使用的快捷方式创建的，系统或自动配置好注册代码。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          pakage="com.example.broadcasttest">
    <uses-permission android:name="android.permission.ACCES_NETWORK_STATE" />
    
        <application
        	tools:replace="android:label,android:theme,android:icon"
        	android:allowBackup="true" 
        	android:icon="@mipmap/ic_launcher"
        	android:label="@string/app_name"
        	android:supportsRtl="true"
        	android:theme="@style/AppTheme" >
    
        	<receiver android:name=".BootCompleteReceiver"
              		  android:enabled="ture"
              		  android:exported="ture" >
    		</receiver>
        </application>
</manifest>
```

`<receiver>` 的标签表示所有静态广播接收器都是在这里进行注册的。但是现在还是不能够接收到开机广播，还需要修改 AndroidManifest.xml 文件。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          pakage="com.example.broadcasttest">
    <uses-permission android:name="android.permission.ACCES_NETWORK_STATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    
        <application
        	tools:replace="android:label,android:theme,android:icon"
        	android:allowBackup="true" 
        	android:icon="@mipmap/ic_launcher"
        	android:label="@string/app_name"
        	android:supportsRtl="true"
        	android:theme="@style/AppTheme" >
    
        	<receiver android:name=".BootCompleteReceiver"
              		  android:enabled="ture"
              		  android:exported="ture" >
                <intent-filter>
                	<action android:name="android.intent.action.BOOT_COMPLETED" />
                </intent-filter>
    		</receiver>
        </application>
</manifest>
```

由于 Android 系统启动后会发出一条值为 `android.intent.action.BOOT_COMPLETED` 的广播，因此我们在 `<intent-filter>` 标签里添加响应的 action。另外，监听系统开机广播也是需要声明权限的，所以使用了 `<uses-permission>` 标签又加入了一条 android.permission.RECEIVE_BOOT_COMPLETED 权限。

重新运行程序后，程序就能接收到开机广播了。注意不要再 onReceive() 方法中添加过多的逻辑或者进行任何的耗时操作，因为广播接收器中是不允许开启线程的，当 onReceive() 方法运行了较长事件而没有结束时，程序就会报错。因此广播接收器更多的是扮演一种打开程序其它组件的角色。



## 发送自定义广播



### 发送标准广播

在发送广播之前，我们还是需要先定义一个广播接收器来准备接收器来准备接收此广播才行，不然发出去也是白发。因此新建一个 MyBroadcastReceiver.java， 代码如下所示：

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        // 弹出消息框
        Toast.makeText(context, "Received in MyBroadcastReceiver", 
                       Toast.LENGTH_LONG).show();
    }
}
```

这里当 MyBroadcastReceiver 收到自定义的广播时就会弹出 “Received in MyBroadcastReceiver” 的提示。然后在 AndroidManifest.xml 中对这个广播接收器进行修改：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          pakage="com.example.broadcasttest">
    <uses-permission android:name="android.permission.ACCES_NETWORK_STATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    
        <application
        	tools:replace="android:label,android:theme,android:icon"
        	android:allowBackup="true" 
        	android:icon="@mipmap/ic_launcher"
        	android:label="@string/app_name"
        	android:supportsRtl="true"
        	android:theme="@style/AppTheme" >
    
        	<receiver android:name=".MyBroadcastReceiver"
              		  android:enabled="ture"
              		  android:exported="ture" >
                <intent-filter>
                	<action android:name="com.example.broadcasttest.MY_BROADCAST" />
                </intent-filter>
    		</receiver>
        </application>
</manifest>
```

可以看到，这里让 MyBroadcastReceiver 接收一条值为 com.example.broadcasttest.MY_BROADCAST 的广播，待会发送广播的时候就需要这样的一条广播。

接下来修改 activity_main.xml 中的代码

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
		android:text="Send Broadcast"
        />
</LinearLaout>
```

这里在布局文件中定义了一个按钮，作为发送广播的触发点。然后修改 MainActivity.java 中的代码，如下所示：

```java
public class MainActivity extends AppCompatActivity {
    // ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = // 构建一个 Intent 对象 并传值进去
                    new Intent("com.example.broadcasttest.MY_BROADCAST");
                // 发送标准广播
                sendBroadcast(intent);
            }
        });
        // ...
    }
    
    // ...
}
```

运行程序，点击按钮就能看到广播的响应事件被触发了。

广播是一种可以跨进程的通信方式，因此某个应用程序内发出的广播，其它应用程序应该也可以收到的。为了验证这一点，再新建一个 BroadcastTest2 项目，创建好项目之后，需要在项目下定义一个广播接收器，用于接收上一小节中的自定义广播。新建 AnotherBroadcastReceiver，代码如下

BroadcastTest2  -> AnotherBroadcastReceiver.java

```java
public class AnotherBroadcastReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        // 弹出消息框
        Toast.makeText(context, "Received in AnotherBroadcastReceiver", 
                       Toast.LENGTH_LONG).show();
    }
}
```

BroadcastTest2  -> AndroidManifest.xml 

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"          
          pakage="com.example.broadcasttest2">    
    <uses-permission android:name="android.permission.ACCES_NETWORK_STATE" />    
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" /> 
    
    <application tools:replace="android:label,android:theme,android:icon"        	
                 android:allowBackup="true"         	
                 android:icon="@mipmap/ic_launcher"        	
                 android:label="@string/app_name"        	
                 android:supportsRtl="true"        	
                 android:theme="@style/AppTheme" >           
        <receiver android:name=".AnotherBroadcastReceiver"              		  
                  android:enabled="ture"              		  
                  android:exported="ture" >                
            <intent-filter>                	
                <action android:name="com.example.broadcasttest.MY_BROADCAST" />                </intent-filter>    		
        </receiver>        
    </application>
</manifest>
```

将两个程序都安装到设备上并运行，点击按钮就会分别弹出两次提示信息，这就证明了广播是可以被其他的应用程序接收到的。到目前为止我们发送的还都是标准广播。



### 发送有序广播

重新回到 BroadcastTest 项目，尝试发送有序广播，修改 MainActivity.java 中的代码如下：

BroadcastTest  -> MainActivity.java

```java
public class MainActivity extends AppCompatActivity {
    // ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = // 构建一个 Intent 对象 并传值进去
                    new Intent("com.example.broadcasttest.MY_BROADCAST");
                // ！-----------发送有序广播-----------！
                sendOrderedBroadcast(intent， null);
            }
        });
        // ...
    }
    
    // ...
}
```

只需改动一行代码，即将 sendBroadcast() 改成 sendOrderedBroadcast() 方法，其中第二个参数是与权限相关的字符串，这里输入 null 就可以。运行程序仍能看到之前一样的效果。

接下来验证广播接收的先后顺序。修改 BroadcastTest 项目的 AndroidManifest.xml 

BroadcastTest  -> AndroidManifest.xml 

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"          
          pakage="com.example.broadcasttest">    
    <uses-permission android:name="android.permission.ACCES_NETWORK_STATE" />    
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" /> 
    
    <application tools:replace="android:label,android:theme,android:icon"        	
                 android:allowBackup="true"         	
                 android:icon="@mipmap/ic_launcher"        	
                 android:label="@string/app_name"        	
                 android:supportsRtl="true"        	
                 android:theme="@style/AppTheme" >           
        <receiver android:name=".MyBroadcastReceiver"              		  
                  android:enabled="ture"              		  
                  android:exported="ture" >                
            <intent-filter android:priority="100">                	
                <action android:name="com.example.broadcasttest.MY_BROADCAST" />                </intent-filter>    		
        </receiver>        
    </application>
</manifest>
```

可以通过 `android:priority` 属性给广播接收器设置优先级。这里将 MyBroadcastReceiver 的优先级设置为了 100，以保证它一定会在 AnotherBroadcastReceiver 之前收到广播。然后就是 MyBroadcastReceiver 有权利选择是否继续传递，如果在接收器中添加 `abortBroadcast()` 就会截断这条广播，后面的广播接收器将无法再接收到这条广播。



## 使用本地广播

前面发送和接收的广播全部属于系统全局广播，即发出的广播可以被其它任何应用程序接收到，并且我们也可以接收到来自于其它任何应用程序的广播。这样就容易引起安全性的问题。

为了能够简单地解决广播的安全性问题，Android 引入了一套本地广播机制，使这个机制发出的广播只能够在应用程序内部进行传递，并且广播接收器也只接收来自本应用程序发出的广播。

本地广播主要就是使用了一个 LocalBroadcastManager 来对广播进行管理，并提供了发送广播和注册广播接收器的方法。下面是用法示例，在 MainActivity.java 中修改代码。

```java
public class MainActivity extends AppCompatActivity {
    private IntentFilter inteneFilter;
    
    private LocalReceiver localReceiver;
    private LocalBroadcastManager localBroadcastManager;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        localBroadcastManager = // 获取实例
            LocalBroadcastManager.getInstance(this);
        
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new 
                    Intent("com.example.broadcasttest.LOCAL_BROADCAST");
                // 发送本地广播
                localBroadcastManager.sendBroadcast(intent);
            }
        });
        intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.broadcasttest.LOCAL_BROADCAST");
        localReceiver = new LocalReceiver();
        // 注册本地广播监听器
        localBroadcastManager.registerReceiver(localReceiver, intentFilter);       
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        localBroadcastManager.unregisterReceiver(localReceiver); 
    }
    class LocalReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "Received in local broadcast", 
                       Toast.LENGTH_LONG).show();
        }
    }
}
```

上述代码和动态注册广播接收器以及发送广播的代码基本一致，只不过首先是通过 LocalBroadcastManager 的 getInstance() 党发得到实例。现在运行程序点击按钮就能看到本地广播的响应事件。



## 广播实践——实现强制下线

强制下线功能应该算是比较常见的，很多程序都具备这个功能，例如 QQ 号在别处登录了，就会将你强制下线。简单的强制下线思路，只需要在界面上弹出一个对话框，让用户无法进行操作，必须点击对话框中的确认按钮，然后回到登录界面即可。

思考：当用户被强制下线的时候可能正处于任何一个界面，难道每个界面都要写一个弹出对话框的逻辑？其实不用这么复杂，这个功能完全可以通过广播实现。新建一个项目 BroadcastBestPractice 项目。

```
BroadcastBestPractice  
	 ├ app/src/main
	 |          ├ java/com... 
	 |          |       ├ MainActivity.java
	 |          |       ├ ActivityCollector.java
	 |          |       ├ BaseActivity.java
	 |          |       ├ .java	             
	 |          |       ├ .java	             
	 |          |       └ .java
	 |          ├ res ┬ layout 
	 |          |     |    ├ activty_main.xml 
	 |          |     |    ├ .xml 
	 |          |     |    ├ .xml 
	 |          |     |    ├ .xml 
	 |          |     |    └ .xml 	 
	 |          |     └ ...
	 |          ├ AndroidManifest.xml
	 |          ├ build.gradle			 
	 |          └ ...
     └ ...
```

首先强制下线功能需要先关闭所有活动，然后回到登录界面。创建一个 ActivityCollector.java 类用于管理所有的活动。

ActivityCollector.java

```java
public class ActivityCollector {
    public static List<Activity> activities = new ArrayList<>();
    
    public static void addActivity(Activity activity){
        activities.add(activity);
    }
    
    public static void removeActivity(Activity activity){
        activities.remove(activity);
    }
    
    public static void finishAll() {
        for (Activity activity : activities) {
            if (!activity.isFinishing()) {
                activity.finish();
            }
        }
    }
    
}
```

然后创建所有活动的父类 BaseActivity.java

```java
public class public class BaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityCollector.addActivity(this);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        ActivityCollector.removeActivity(this);
    }
} extends AppCompatActivity {
    
}
```

新建 LoginActivity，并让IDE帮助我们自动生成相应的布局文件 activity_login.xml

```xml

<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".ui.login.LoginActivity">

    <EditText
        android:id="@+id/username"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="24dp"
        android:layout_marginTop="96dp"
        android:layout_marginEnd="24dp"
        android:hint="@string/prompt_email"
        android:inputType="textEmailAddress"
        android:selectAllOnFocus="true"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/password"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="24dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="24dp"
        android:hint="@string/prompt_password"
        android:imeActionLabel="@string/action_sign_in_short"
        android:imeOptions="actionDone"
        android:inputType="textPassword"
        android:selectAllOnFocus="true"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/username" />

    <Button
        android:id="@+id/login"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="start"
        android:layout_marginStart="48dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="48dp"
        android:layout_marginBottom="64dp"
        android:enabled="false"
        android:text="@string/action_sign_in"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/password"
        app:layout_constraintVertical_bias="0.2" />

    <ProgressBar
        android:id="@+id/loading"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_marginStart="32dp"
        android:layout_marginTop="64dp"
        android:layout_marginEnd="32dp"
        android:layout_marginBottom="64dp"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="@+id/password"
        app:layout_constraintStart_toStartOf="@+id/password"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.3" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

接下来修改 LoginActivity.java

```java
public class LoginActivity extends BaseActivity {

    private LoginViewModel loginViewModel;
    private ActivityLoginBinding binding;
	// ...
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityLoginBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        loginViewModel = new ViewModelProvider(this, new LoginViewModelFactory())
                .get(LoginViewModel.class);

        final EditText usernameEditText = binding.username;
        final EditText passwordEditText = binding.password;
        final Button loginButton = binding.login;
        final ProgressBar loadingProgressBar = binding.loading;

        loginButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loadingProgressBar.setVisibility(View.VISIBLE);

                String name = usernameEditText.getText().toString();
                String password = passwordEditText.getText().toString();

                if (name.equals("admin") && password.equals("123456")) {
                    Intent intent = 
                        new Intent(LoginActivity.this, MainActivity.class);
                    startActivity(intent);
                    finish();
                } else {
                    Toast.makeText(LoginActivity.this, "name or password is invalid",
                            Toast.LENGTH_SHORT).show();
                }

            }
        });
    }

// ...
}
```

修改 activity_main.xml 

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/force_offline"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send force offline broadcast" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

修改 MainActivity.java

```java
public class MainActivity extends BaseActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button forceOffline = (Button) findViewById(R.id.force_offline);
        forceOffline.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("com.example.broadcastbest.FORCE_OFFLINE");
                sendBroadcast(intent);
            }
        });

    }
}
```

修改 BaseActivity.java

```java
public class BaseActivity extends AppCompatActivity {

    private ForceOffLineReceiver receiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityCollector.addActivity(this);
    }

    @Override
    protected void onResume() {
        super.onResume();
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.broadcastbest.FORCE_OFFLINE");
        receiver = new ForceOffLineReceiver();
        registerReceiver(receiver, intentFilter);
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (receiver != null) {
            unregisterReceiver(receiver);
            receiver = null;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        ActivityCollector.removeActivity(this);
    }

    class ForceOffLineReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(final Context context, Intent intent) {
            AlertDialog.Builder builder = new AlertDialog.Builder(context);
            builder.setTitle("Warning");
            builder.setMessage("You are forced ....");
            builder.setCancelable(false);
            builder.setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    ActivityCollector.finishAll();
                    Intent intent = new Intent(context, LoginActivity.class);
                    context.startActivity(intent);
                }
            });
            builder.show();
        }
    }
}

```

修改 AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.broadcastbest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BroadcastBest">

        <activity android:name=".MainActivity">

        </activity>
        <activity android:name=".ui.login.LoginActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```



# 7 数据存储全方案

【数据存储全方案】详解持久化技术





## 持久化技术简介

数据的持久化就是指将那些内存中的瞬时数据保存到存储设备中，保证即使在手机或电脑关机的情况下，这些数据仍然不会丢失。保存在内存中的数据是处于瞬时状态的，而保存在存储设备中的数据是处于持久状态的，持久化技术则提供了一种机制可以让数据在瞬时状态和持久状态之间进行转换。

持久化技术被广泛应用于各种程序设计的领域当中，而本书中要探讨的自然是 Android 中的数据持久化技术。Android 系统中主要提供了 3 种方式用于简单地实现数据持久化功能，即文件存储、SharedPreference 存储以及数据库存储。当然，除了这 3 种方式之外，你还可以将数据保存 在手机的 SD 卡中，不过使用文件、SharedPreference 或数据库来保存数据会相对更简单一些，而且比起数据保存在 SD 卡中会更加地安全。



## 文件存储



文件存储是 Android 中最基本的一种数据存储方式，它不对存储的内容进行任何格式化处理，所有数据都是原封不动地保存到文件中的，因此它比较适合用于存储一些简单的文本数据或二进制数据。如果想使用文件存储的方式保存一些较为复杂的文本数据就需要定义一套自己的格式规范，这样可以方便之后将数据从文件中解析出来。

### 将数据存储到文件中

Content 类中提供了一个 openFileOutput() 方法，可以用于将数据存储到指定的文件中。这个方法接收两个参数，第一个参数是文件名，在文件创建的时候使用的就是这个名称，注意这里的文件名可以不包含路径，因为所有的文件都是默认存储到 `/data/data/<packagename>/files/` 目录下的。第二个参数是文件的操作模式，主要有两种模式可选，`MODE_PRIVATE` 和 `MODE_APPEND` 。其中 `MODE_PRIVATE` 是默认的操作模式，表示当指定同样文件名的时候，所写入的内容将会覆盖原文件中的内容，`MODE_APPEND` 则表示如果该文件已经存在，就往文件里面追加内容，不存在就创建新文件。

openFileOutput() 方法返回的是一个 FileOutputStream 对象，得到了这个对象之后可以使用 Java 流的方式将数据写入到文件中了。下面是一段简单的代码示例，展示了如何将一段文本内容保存到文件中：

```java
public void save() {
    String data = "Data to save";
    FileOutputStream out = null;
    BufferedWriter writer = null;
    try {
        out = openFileOutput("data", Context.MODE_PRIVATE);
        Writer = new BufferedWriter(new OutputStreamWriter(out));
        writer.write(data);
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            if (writer != null) {
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```



### 从文件中读取数据

类似于将数据存储到文件中，Content 类中还提供了一个 openFileInput() 方法，用于从文件中读取数据。它只接收一个参数，即要读取的文件名，然后系统会自动到 `/data/data/<packagename>/files/` 目录下去加载这个文件，并返回一个 FileInputStream 对象，得到了这个对象之后再通过 Java 流的方式就可以将数据读取出来了。

下面是一段简单的代码示例，展示了如何从文件中读取文本数据：

```java
public void load() {
    
    FileInputStream in = null;
    BufferedReader reader = null;
    StringBuilder content = new StringBuilder();
    try {
        in = openFileIntput("data");
        reader = new BufferedReader(new InputStreamReader(in));
        String line = "";
        while ((line = reader.readLine()) != null){
            content.append(line);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            if (reader != null) {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    return content.toString();
}
```





## SharedPreferences 存储

不同于文件的存储方式，SharedPreferences 是使用键值对的方式来存储数据的。也就是说，当保存一条数据的时候，需要给这条数据提供一个对应的键，这样在读取数据的时候就可以通过这个键把相应的值取出来。而且 SharedPreferences 还支持多种不同的数据类型存储，如果存储的数据类型是整型，那么读出来的也是整型；如果存储的数据类型是字符串，那么读出来的也是字符串。



### 将数据存储到 SharedPreferences 中

想要使用 SharedPreferences 来存储数据，首先需要获取到 SharedPreferences 对象。Android 中主要提供了3种方法用于得到 SharedPreferences 对象。

1. **Content 类中的 getSharedPreferences() 方法**

此方法接收两个参数，第一个参数用于指定 SharedPreferences 文件的名称，如果指定的文件不存在则会创建一个，SharedPreferences 文件都是存放在 `/data/data/<packagename>/shared_prefs/` 目录下的。第二个参数用于指定操作模式，目前只有 MODE_PRIVATE 这一种模式可选，它是默认的操作模式，和直接传入 0 效果是相同的，表示只有当前的应用程序才可以对这个 SharedPreferences 文件进行读写。

2. **Activity 类中的 getPreferences() 方法**

这个方法和 Context 中的 getSharedPreferences() 方法很相似，不过它只接收一个操作模式参数，因为使用这个方法时会自动将当前活动的类名作为 SharedPreferences 的文件名。

3. **PreferenceManager 类中的 getDefaultSharedPreferences() 方法**

这是一个静态方法，它接收一个 Context 参数，并自动使用当前应用程序的包名作为前缀来命名 SharedPreferences 文件，得到 SharedPreferences 对象之后就可以开始向 SharedPreferences 文件中存储数据了。

向 SharedPreferences 文件中存储数据主要分为一下三步。

1. 调用 SharedPreferences 对象的 edit() 方法来获取一个 SharedPreferences.Editor 对象。
2. 向 SharedPreferences.Editor 对象中添加数据，例如，添加一个布尔型数据就使用 putBoolean() 方法调用，添加一个字符串就使用 putString() 方法。
3. 调用 apply() 方法将添加的数据提交，从而完成数据存储操作。



代码案例：新建一个 SharedPreferencesTest 项目，然后修改 activity_main.xml 中的代码（添加一个按钮）

```xml
<LinearLaout xmlnx:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
    android:layout_width="macth_parent"
    android:layout_height="macth_parent">
	
    <Button
    	android:id="@+id/save_data"
        android:layout_width="match_parent"
        android:layout_height="warp_content"
		android:text="Save data"
        />
</LinearLaout>
```

然后修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstaceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button saveData = (Button) findViewById(R.id.save_data);
        saveData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SharedPreferences.Editor editor = 
                	getSharedPreferences("data", MODE_PRIVATE).edit();
                editor.putString("name", "Tom");
                editor.putInt("age", 18);
                editor.putBoolean("married", false);
                editor.apply();            
            }
  
        });
    }
}
```

运行程序，即可实现将数据存储到 SharedPreferences 中。

### 从 SharedPreferences 中读取数据

读取数据，相对写入数据都是对称操作，获取对象的方法都相同，只是将所有 put 替换成 get，get 的第二个参数表示没有找到对应的数据，就会用默认的数据（第二个参数填入的内容）

```java
public class MainActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstaceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button restoreData = (Button) findViewById(R.id.restore_data);
        restoreData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SharedPreferences pref = 
                	getSharedPreferences("data", MODE_PRIVATE);
                String name = pref.putString("name", "");
                int age = pref.putInt("age", 0);
                boolean married = pref.putBoolean("married", false);
                Log.d("MainActivity", "name is" + name);
                Log.d("MainActivity", "age is" + age);
                Log.d("MainActivity", "married is" + namarried);
            }
  
        });
    }
}
```





### 实现记住密码功能

在前面的 BroadcastBest 项目的基础上，添加一个记住密码的功能

```
BroadcastBest
	 ├ app/src/main
	 |          ├ java/com... 
	 |          |       ├ MainActivity.java
	 |          |       ├ ActivityCollector.java
	 |          |       ├ BaseActivity.java
	 |          |       ├ .java	             
	 |          |       ├ .java	             
	 |          |       └ .java
	 |          ├ res ┬ layout 
	 |          |     |    ├ activty_main.xml 
	 |          |     |    ├ .xml 
	 |          |     |    ├ .xml 
	 |          |     |    ├ .xml 
	 |          |     |    └ .xml 	 
	 |          |     └ ...
	 |          ├ AndroidManifest.xml
	 |          ├ build.gradle			 
	 |          └ ...
     └ ...
```



首先编辑一下登录界面的布局，打开 activity_login.xml 中的代码，

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".ui.login.LoginActivity">


    <!-- ...... -->
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        tools:ignore="MissingConstraints">
        <CheckBox
            android:id="@+id/remember_pass"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:text="Remember password" />
    </LinearLayout>

    <Button
        android:id="@+id/login"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="start"
        android:layout_marginStart="48dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="48dp"
        android:layout_marginBottom="64dp"
        android:enabled="false"
        android:text="@string/action_sign_in"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/password"
        app:layout_constraintVertical_bias="0.2" />

    <!-- ...... -->
</androidx.constraintlayout.widget.ConstraintLayout>
```

修改 LoginActivity.java

```java
public class LoginActivity extends BaseActivity {

    private LoginViewModel loginViewModel;
    private ActivityLoginBinding binding;

    private SharedPreferences pref;
    private SharedPreferences.Editor editor;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityLoginBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        loginViewModel = new ViewModelProvider(this, new LoginViewModelFactory())
                .get(LoginViewModel.class);

        final EditText usernameEditText = binding.username;
        final EditText passwordEditText = binding.password;
        final Button loginButton = binding.login;
        final ProgressBar loadingProgressBar = binding.loading;

        final CheckBox rememberPassCheckBox = (CheckBox)findViewById(R.id.remember_pass);

        pref = PreferenceManager.getDefaultSharedPreferences(this);
        boolean isRemember = pref.getBoolean("remember_password", false);
        if (isRemember) {
            String name = pref.getString("name", "");
            String password = pref.getString("password", "");
            usernameEditText.setText(name);
            passwordEditText.setText(password);
            rememberPassCheckBox.setChecked(true);
        }

        // ...
       
        loginButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loadingProgressBar.setVisibility(View.VISIBLE);
//                loginViewModel.login(usernameEditText.getText().toString(),
//                        passwordEditText.getText().toString());
                String name = usernameEditText.getText().toString();
                String password = passwordEditText.getText().toString();

                if (name.equals("admin") && password.equals("123456")) {
                    editor = pref.edit();
                    if (rememberPassCheckBox.isChecked()) {
                        editor.putBoolean("remember_password", true);
                        editor.putString("name", name);
                        editor.putString("password", password);
                    } else {
                        editor.clear();
                    }
                    editor.apply();
                    
                    Intent intent = new Intent(LoginActivity.this, MainActivity.class);
                    startActivity(intent);
                    finish();
                } else {
                    Toast.makeText(LoginActivity.this, "name or password is invalid",
                            Toast.LENGTH_SHORT).show();
                }

            }
        });
    }


}
```

这样就实现了记住密码的功能。



## SQLite 数据库存储



Android 系统是内置数据库的，SQLite 是一款轻量级的关系型数据库，它的运算速度非常快，占用资源非常少，通常只需要几百 KB 的内存就足够了，因而特别适合在移动设备上使用。SQLite 不仅支持标准的 SQL 语法，还遵循了数据库的 ACID 事务，所以只要你以前使用过其他的关系型数据库，就能很快上手 SQLite。而且 SQLite 又比一般的数据库要简单的多，它甚至不用设置用户名和密码就可以使用。



### 创建数据库

Android 为了让我们更加方便地管理数据，专门提供了一个 SQLiteOpenHelper 帮助类。借助这个类就可以非常简单地对数据可进行创建和升级。

SQLiteOpenHelper 是一个抽象类，如果我们要使用它的话，就需要创建一个自己的帮助类去继承它。SQLiteOpenHelper 中有两个抽象方法，分别是 onCreate() 和 onUpgrade() ，我们必须在自己的帮助类里面重写这两个方法，然后这两个方法中去实现创建、升级数据库的逻辑。

SQLiteOpenHelper 中还有两个非常重要的实例方法：getReadableDatabase() 和 getWritableDatabase() 。这两个方法都可以创建或打开一个现有的数据库（如果数据库已存在则直接打开，否则创建一个新数据库），并返回一个可对数据库进行读写操作的对象。不同的是当数据库不可写入时（如磁盘已满）getReadableDatabase 返回的对象以只读方式打开数据库，而 getWritableDatabase 将出现异常。



SQLiteOpenHelper 中有两个构造方法可供重写，一般使用参数少一点的那个构造方法即可。这个构造方法中接收 4 个参数，第一个参数是 Context ，必须有上下文才能对数据库进行操作，第二个是数据库名，创建数据库时使用的就是这里指定的名称。第三个参数允许我们在查询数据的时候返回一个自定义的 Cursor， 一般都是传入 null。第四个参数表示当前数据库的版本号，可用于对数据库进行升级操作。构建出 SQLiteOpenHelper 的实例后，再调用它的 getReadableDatabase() 或 getWritableDatabase() 方法就能够创建数据库了，数据库文件会存放在 `/data/data/<package name>/databases` 目录下。此时，重写的 onCreate() 方法也会得到执行，所以通常会在这里去处理一些创建表的逻辑。接下来首先见一个 DatabaseTest 项目。

这里希望创建一个名为 BookStore.db 的数据库，然后在这个数据中新建一张 Book 表，表中有 id（主键）、作者、价格、页数和书名等列。Book 表的建表语句如下所示：

```sql
create table Book (
	id integer primary key autoincrement,
	author text,
	price real,
	pages integer,
	name text)
```

然后需要在代码中执行这条 SQL 语句，才能完成创建表的操作。新建 MyDatabaseHelper 类继承自 SQLiteOpenHelper ，代码如下

```java
public class MyDatabaseHelper extends SQLiteOpenHelper {
    public static final String CREATE_BOOK = "create table Book (" +
            "id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)";
    private Context mContext;

    public MyDatabaseHelper(Context context, String name,
                            SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        mContext = context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_BOOK);
        Toast.makeText(mContext, "Create succeeded", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){

    }

}

```

最后修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {

    private MyDatabaseHelper dbHelper;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 1);
        Button createDatabase = (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
        });

    }
}
```

运行程序，第一次点击按钮可以看见弹出提示框，之后再点击就没有提示了。这是因为第一次创建成功后，再点击就不会重复创建了。我们可以使用 adb shell 查看 /data/data/\<package name\>/database/ 路径下有两个数据库文件，一个是我们创建的 BookStore.db，另一个 BookStore.db-journal 则是为了让数据库能够支持事务而产生的临时日志文件。



### 升级数据库

onUpgrade() 方法是用于对数据库进行升级的，它在整个数据库的管理工作中起着非常重要的作用。

目前 DatabaseTest 项目中已经有一张 Book 表用于存放书的各种详细数据，如果我们想再添加一张 Category 表用于记录图书的分类，比如 Category 表中有 id（主键）、分类名和分类代码这几个列，那么建表语句就可以写成：

```sql
create table Category (
	id integer primary key autoincrement,
    category_name text,
    category_code integer)
```

接下来将这条语句添加到 MyDatabaseHelper 中

```java
public class MyDatabaseHelper extends SQLiteOpenHelper {
    public static final String CREATE_BOOK = "create table Book (" +
            "id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)";

    public static final String CREATE_CATEGORY = "create table Category (" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)";
    private Context mContext;

    public MyDatabaseHelper(Context context, String name,
                            SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        mContext = context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_BOOK);
        db.execSQL(CREATE_CATEGORY);
        Toast.makeText(mContext, "Create succeeded", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){

    }

}

```

这时候运行程序，点击按钮并没有弹出提示框，说明没有创建新的数据库，这是因为 BookStore.db 数据库已经存在了，再点击按钮，MyDatabaseHelper 中的 onCreate() 方法就不会执行，因此新添加的表就无法得到创建了。

解决这个问题的一个办法就是把程序先卸载掉再重新安装运行，但是 我们可以通过重写 onUpgrade() 方法实现数据库升级，修改 MyDatabaseHelper.java 中的代码。

```java
public class MyDatabaseHelper extends SQLiteOpenHelper {
    public static final String CREATE_BOOK = "create table Book (" +
            "id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)";

    public static final String CREATE_CATEGORY = "create table Category (" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)";
    private Context mContext;

    public MyDatabaseHelper(Context context, String name,
                            SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        mContext = context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_BOOK);
        db.execSQL(CREATE_CATEGORY);
        Toast.makeText(mContext, "Create succeeded", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){
        db.execSQL("drop table if exists Book");
        db.execSQL("drop table if exists Category");
        onCreate(db);

    }

}
```

可以看到，在 onUpgrade 方法中执行了两条 DROP 语句，如果发现数据库中已经存在 Book 表或 Category 表了，就将这两张表删除掉，然后再调用 onCreate 方法重新创建。因为如果在创建表时发现这张表已经存在了，就会直接报错。

接下来就是如何让 onUpgrade() 执行起来，我们只需要在 SQLiteOpenHelper 构造方法的第四个参数传入一个比 1 大的数字就可以了。



### 添加数据（增）

这一部分将介绍对数据的操作，主要分为 4 种，即 CRUD。C 代表添加（Create），R 代表查询（Retrieve），U 代表更新（Updata），D 代表（Delete）。

每一种操作又各自对应了一种 SQL 命令，Android 为了方便编写，提供了一系列辅助性的方法，即使不用编写 SQL 语句，也能完成所有的 CRUD 操作。

调用 getReadableDatabase() 和 getWritableDatabase() 可以收到一个 SQLiteDatabase 对象，SQLiteDatabase 中提供了一个 insert() 这个方法就是专门用于添加数据。它就收 3 个参数，第一个参数是表名，第二个参数用于在未定添加数据的情况下给某些可为空的列自动赋值 NULL，一般我们用不到这个功能，直接传入 null 即可。第三个参数是一个 ContentValues 对象，它提供了一系列的 put() 方法重载，用于向 ContentValues 中添加数据，只需要将表中的每个列名以及相应的待添加数据传入即可。

先修改 activity_main.xml 中的代码，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        tools:ignore="MissingConstraints">
        <Button
            android:id="@+id/create_database"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="Create Database"
            tools:ignore="MissingConstraints" />

        <Button
            android:id="@+id/add_data"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Add data"
            tools:ignore="MissingConstraints" />
    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

接着就修改 MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private MyDatabaseHelper dbHelper;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2);
        Button createDatabase = (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
        });

        Button addData = (Button) findViewById(R.id.add_data);
        addData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();

                values.put("name", "The Da Vinci Code");
                values.put("author", "Dan Brown");
                values.put("pages", 454);
                values.put("price", 16.96);
                db.insert("Book", null, values);
                values.clear();

                values.put("name", "The Lost symbol");
                values.put("author", "Dan Brown");
                values.put("pages", 510);
                values.put("price", 19.95);
                db.insert("Book", null, values);
            }
        });

    }
}
```

再次运行程序，点击添加数据按钮，然后使用 adb 工具，先 adb root 再 adb remount 最后使用 adb pull /data/data/com.databasetest/databases/BookStore.db 把数据库从 Android 拉到本地上，使用 SQLite Expert 查看，确认有正确的数据插入到 Book 表中。



### 更新数据（改）

SQLiteDatabase 中提供一个非常好用的方法 updata() ，用于对数据进行更新，这个方法接收 4 个参数，第一个参数是表名，第二个参数是 ContentValues 对象，要把更新数据在这里组装进去。第三第四个参数用于约束更新某一行或某几行中的数据，不指定的话默认就是更新所有行。

接下来在 DatabaseTest 项目的基础上修改，看一下更新数据的具体用法。如要调整第一本书的价格。首先修改 activity_main.xml 中的代码，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        tools:ignore="MissingConstraints">
        <Button
            android:id="@+id/create_database"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="Create Database"
            tools:ignore="MissingConstraints" />

        <Button
            android:id="@+id/add_data"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Add data"
            tools:ignore="MissingConstraints" />
        <Button
            android:id="@+id/update_data"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Update data"
            tools:ignore="MissingConstraints" />
    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

接着就修改 MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private MyDatabaseHelper dbHelper;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2);
        Button createDatabase = (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
        });

        Button addData = (Button) findViewById(R.id.add_data);
        addData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();

                values.put("name", "The Da Vinci Code");
                values.put("author", "Dan Brown");
                values.put("pages", 454);
                values.put("price", 16.96);
                db.insert("Book", null, values);
                values.clear();

                values.put("name", "The Lost symbol");
                values.put("author", "Dan Brown");
                values.put("pages", 510);
                values.put("price", 19.95);
                db.insert("Book", null, values);
            }
        });

        Button update = (Button) findViewById(R.id.update_data);
        update.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();
                values.put("price", 10.9);
                db.update("Book", values, "name = ?", new String[] {"The Da Vinci Code"});
            }
        });

    }
}
```

重新运行程序，按照上一个方法查看数据库，就能看到数据更新的效果了。



### 删除数据（删）

SQLiteDatabase 中提供一个 delete() 方法，用于删除数据，这个方法接收三个参数，第一个参数是表名，第二个第三个参数是约束删除某一行或某几行的数据，不指定的话是默认删除所有行。

首先修改 activity_main.xml 中的代码，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        tools:ignore="MissingConstraints">
        <Button
            android:id="@+id/create_database"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="Create Database"
            tools:ignore="MissingConstraints" />

        <Button
            android:id="@+id/add_data"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Add data"
            tools:ignore="MissingConstraints" />
        <Button
            android:id="@+id/update_data"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Update data"
            tools:ignore="MissingConstraints" />
        <Button
            android:id="@+id/delete_data"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Delete data"
            tools:ignore="MissingConstraints" />
    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

接着就修改 MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private MyDatabaseHelper dbHelper;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2);
        Button createDatabase = (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
        });

        Button addData = (Button) findViewById(R.id.add_data);
        addData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();

                values.put("name", "The Da Vinci Code");
                values.put("author", "Dan Brown");
                values.put("pages", 454);
                values.put("price", 16.96);
                db.insert("Book", null, values);
                values.clear();

                values.put("name", "The Lost symbol");
                values.put("author", "Dan Brown");
                values.put("pages", 510);
                values.put("price", 19.95);
                db.insert("Book", null, values);
            }
        });

        Button update = (Button) findViewById(R.id.update_data);
        update.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();
                values.put("price", 10.9);
                db.update("Book", values, "name = ?", new String[] {"The Da Vinci Code"});
            }
        });

        Button deleteButton = (Button) findViewById(R.id.delete_data);
        deleteButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                db.delete("Book", "pages > ?", new String[] {"500"});
            }
        });

    }
}
```

重新运行程序，按照上一个方法查看数据库，就能看到数据更新的效果了。



### 查询数据（查）

SQL 的全称是 Structured Query Language，翻译成中文就是结构化查询语言。它的大部分功能都体现在 ”查“ 这个字上的，由于 SQL 查询涉及的内容实在是太多了，这里只介绍 Android 上的查询功能。

SQLiteDatabase 中提供一个 query() 方法，用于对数据进行查询。这个方法的参数非常复杂，最短的一个方法也需要传入 7 个参数。第一个是表名；第二个参数用于指定去查询哪几列，如果不指定则默认查询所有列。第三、第四个参数用于约束查询某一行或某几行的数据，不指定则默认查询所有行的数据。第五个参数用于指定需要去 group by 的列，不指定则表示不对查询结果进行 group by 操作。第六个参数用于对 group by 之后的数据进行进一步的过滤，不指定则表示不进行过滤。第七个参数用于指定查询结果的排序方式，不指定表示使用默认的排序方式。详细内容见下表。



| query()方法参数 |        对应SQL部分        | 描述                          |
| :-------------: | :-----------------------: | :---------------------------- |
|      table      |      from table_name      | 指定查询的表名                |
|     columns     |  select coumn1,  coumn2   | 指定查询的列名                |
|    selection    |   where column = value    | 指定where的约束条件           |
|  selectionArgs  |             -             | 为where中的占位符提供具体的值 |
|     groupBy     |      group by column      | 指定需要group by的列          |
|     having      |   having column = value   | 对group by后的结果进一步约束  |
|     orderBy     | order by column1, column2 | 指定查询结果的排序方式        |

调用 query() 方法后会返回一个 Cursor 对象，查询到所有数据都将从这个对象中取出。下面是使用示例。

```java
Button queryButton = (Button) findViewById(R.id.query_data);
queryButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        Cursor cursor = db.query("Book", null, null, null, 
                                 null, null, null);
        if (cursor.moveToFirst()) {
            do {
                String name = 
                    cursor.getString(cursor.getColumnIndex("name"));
                String author = 
                    cursor.getString(cursor.getColumnIndex("author"));
                int pages = 
                    cursor.getInt(cursor.getColumnIndex("pages"));
                double price = 
                    cursor.getDouble(cursor.getColumnIndex("price"));
            } while (cursor.moveToNext());
        }
    }
});
```

上述就是 SQLite 简单的使用。



### 使用 SQL 操作数据库

这一部分将简略演示，直接使用 SQL 来完成 增删改查。

* 添加数据（增）

```java
db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)", 
          new String[] { "The Da VinciCode", "Dan Brown", "454", "16.96"});
db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)", 
          new String[] { "The Lost Symbol", "Dan Brown", "510", "19.95"});
```

* 更新数据（改）

```java
db.execSQL("update Book set price = ? where name = ?", new String[] { "10.99", 
           "The Da VinciCode" });
```

* 删除数据的方法（删）

```java
db.execSQL("delete from Book where page > ?", new String[] { "500" });
```

* 查询数据（查）

```java
db.execSQL("select * from Book", null);
```





## 使用 LitePal 操作数据库



略





# 8 跨程序共享数据

【跨程序共享数据】探究内容提供器

## 内容提供器简介

内容提供器（Content Provider）主要用于在不同的应用程序之间实现数据共享的功能，它提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性。目前，使用内容提供器是 Android 实现跨程序共享数据的标准方式。

不同于文件存储和 SharedPreferences 存储中的两种全局可读写操作模式，内容提供器可以选择只对哪一部分数据进行共享，从而保证我们程序中的隐私数据不会有泄漏的风险。

在学习内容提供器之前，需要先了解 Android 运行时权限，因为内容提供器示例中会使用到运行时权限的功能。



## 运行时权限

### Android 权限机制详解

首先了解过去的 Android 权限机制是什么样的。在前面写的 BroadcastTest 项目的时候第一次接触了 Android 权限相关内容，当时为了要访问系统的网络状态以及监听开机广播在 AndroidManifest.xml 文件中添加了两句权限声明。

因为访问系统的网络状态以及监听开机广播涉及了用户设备的安全性，因此必须在 AndroidManifest.xml 中加入权限声明，否则我们的程序就会崩溃。加入这两句权限声明后，首先在安装过程中会给用户提醒，另外，用户可以随时在应用程序管理界面查看任意一个程序的权限情况。但是这样有很多应用无论用不用这项功能都会先把权限申请下来。这样对安全造成了影响，也对用户极不方便。

所以后面在 Android 中添加了运行时权限功能。也就是说，用户不需要在安装软件时一次授权所有申请的权限，而是可以在软件的使用过程中再对某一线权限申请进行授权。 而且为了方便操作，Android 现在将所有的权限归成了两类，一类是普通权限，一类是危险权限。普通权限不会直接威胁到用户的安全和隐私的权限，对于这部分权限申请，系统会自动帮我们进行授权。危险权限则表示可能会触及用户隐私，或者对设备安全性造成影响的权限，如获取设备联系人信息、定位设备的地理位置等，对于这部分权限申请，必须要由用户手动点击授权才可以。



### 在程序运行时申请权限

首先新建一个 RuntimePermissionTest 项目，我们就在这个项目的基础上来学习运行时权限的使用方法。这里简单起见我们就使用 CALL_PHONE 这个权限来作为示例。

修改 activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <Button
            android:id="@+id/make_call"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Make Call" />

    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

接着修改 MainActivity.java 中的代码：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button makeCall = (Button)findViewById(R.id.make_call);
        makeCall.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try {
                    Intent intent = new Intent(Intent.ACTION_CALL);
                    intent.setData(Uri.parse("tel:10086"));
                    startActivity(intent);
                } catch (SecurityException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

接着修改 AndroidManifest.xml 文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.runtimepermissiontest">

    <uses-permission android:name="android.permission.CALL_PHONE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.RuntimePermissionTest">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

这时运行程序，点击按钮没有任何反应，查看日志错误提示操作不被允许，因为使用危险权限是都必须进行运行时权限处理。

那么下面就来尝试修复这个问题，修改 MainActivity 中的代码：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button makeCall = (Button)findViewById(R.id.make_call);
        makeCall.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 权限比对 
                if (ContextCompat.checkSelfPermission(MainActivity.this,
                        Manifest.permission.CALL_PHONE)
                != PackageManager.PERMISSION_GRANTED) {
                    // 如果没有 动态申请
                    ActivityCompat.requestPermissions(MainActivity.this, new
                            String[] { Manifest.permission.CALL_PHONE }, 1);
                } else {
                    // 如果有 直接执行
                    call();
                }

            }
        });
    }
    private void call() {
        try {
            Intent intent = new Intent(Intent.ACTION_CALL);
            intent.setData(Uri.parse("tel:10086"));
            startActivity(intent);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 调用完 requestPermissions 会回调 onRequestPermissionsResult 这个方法
    // 授权结果会封装在 grantResults 参数中
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    call();
                } else {
                    Toast.makeText(this, "You denide the permission", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }

}
```

运行程序，点击按钮，允许授权后即可拨打10086。





## 访问其它程序中的数据



### ContentResolver 的基本用法

对于每一个应用程序，如果想要访问内容提供器中共享的数据，就一定要借助 ContentResolver 类，可以通过 Context 中的 getContentResolver() 方法获取到该类的实例。ContentResolver 中提供了一系列的方法用于对数据进行 CRUD 操作，其中 insert() 用于添加数据；update() 用于更新数据；delete() 方法用于删除数据，query() 用于查询数据。

ContentResolver 中的方法跟 SQLiteDatabase 用法类似但是略有不同，ContentResolver 中增删改查的方法都是不接收表名参数的，而是使用一个 Uri 参数代替，这个参数被称为内容 URI。内容 URI 给内容提供器中的数据建立了唯一标识符，它主要由两部分组成：authority 和 path。 authority 是用于对不同的应用程序做区分的，一般为了避免冲突，都会采用程序包名的方式来命名。。例如，某个程序包名是 com.examp.app，那么该程序对应的 authority 就可以命名为 com.examp.app.provider。path 则是用于对同一应用程序中不同的表做区分的，通常会添加到 authority 的后面，比如某个程序的数据库里存在两张表：table1 和 table2 ，这时就可以将 path 分别命名为 /table1 和 /table2 ，然后把 authority 和 path 进行组合，内容 URI 就变成了 com.examp.app.provider/table1 和 com.examp.app.provider/table2。不过，目前还很难辨认出这两个字符串就是两个内容 URI，我们还需要在字符串得头部加上协议声明。因此，内容 URI 最标准的格式写法如下：

```xml
content://com.example.app.provider/table1
content://com.example.app.provider/table2
```

有没有发现，内容 URI 可以非常清楚地表达出我们想要访问哪个程序中哪张表里的数据。也正是因此，ContentResolver 中的增删改查方法才都接收 Uri 对象作为参数，因为如果使用表名的话系统将无法得知我们期望访问的是哪个应用程序里的表。

在得到了内容 URI 字符串之后，我们还需要将它解析成 Uri 对象才可以作为参数传入。解析的方法也很简单

```java
Uri uri = Uri.parse("content://com.example.app.provider/table1")
```

* **查**

现在就可以使用这个 Uri 对象来查询 table 表中的数据，代码如下所示：

```java
cursor cursor = getContentResolver().query(
	uri,
	projection,
	selection,
	seletionArgs,
	sortOrder);
```

查询完成后返回的仍然是一个 Cursor 对象，这时我们就可以将数据从 Cursor 对象中逐个读取出来了。读取的思路仍然是通过移动游标的位置来遍历 Cursor 的所有行，然后再取出每一行中相应的数据。

```java
if (cursor != null) {
    while (cursor.moveToNext()) {
        String column1 = cursor.getString(cursor.getColumnIndex("column1"));
        int column2 = cursor.getInt(cursor.getColumnIndex("column2"));
    }
    cursor.close();
}
```

* **增**

```java
ContentValues values = new ContentValues();values.put("column1", "text");values.put("column2", 1);getContentResolver().insert(uri, values);
```

* **改**

```java
ContentValues values = new ContentValues();values.put("column1", "");getContentResolver().update(uri, values, "column1 = ? and column2 = ?",                           new String[] { "text", "1" });
```

* **删**

```java
getContentResolver().delete(uri, "column2 = ?", new String[] { "1" });
```



### 读取系统联系人

在电话簿里添加几个联系人，新建一个 ContactsTest 项目，首先编写一下布局文件，修改 activity_main.xml 中的代码：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
    		  android:layout_height="match_parent"
              tools:context=".MainActivity">
	
    <ListView android:id="@+id/contacts_view"
              android:layout_width="match_parent"
    		  android:layout_height="match_parent" >
    </ListView>
              
    
</LinearLayout>
```

接着修改 MainActivity.java 

```java
public class MainActivity extends AppCompatActivity {

    ArrayAdapter<String> adapter;
    List<String> contactsList = new ArrayList();
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		ListView contactsView = (ListView) findById(R.id.contacts_view);
        adapter = new ArrayAdapter<String>(this, android.R.layout, 
                                           simple_list_item_1, contactsList);
        contactsView.setAdapter(adapter);
        if (ContextCompat.checkSelfPermission(this, 
        	Manifest.permission.READ_CONTACTS) != 
            PackageManager.PERMISSION_GRANTED) {
            
            ActivityCompat.requestPermissions(this,
            	new String[] { Manifest.permission.READ_CONTACTS }, 1);
        } else {
            readContacts();
        } 
    }
    
    private void readContacts() {
        Cursor cursor = null;
        try {
            // 查询联系人
            cursor = getContentResolver().query(ContactsContract.CommonDataKinds.
            	Phone.CONTENT_URI, null, null, null, null);
            if (cursor != null) {
                while (cursor.moveToNext()) {
                    // 获取联系人姓名
                    String displayName = cursor.getString(
                        cursor.getColumnIndex(
                            ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME)); 
                    // 获取手机号
                    String number = cursor.getString(
                        cursor.getColumnIndex(
                            ContactsContract.CommonDataKinds.Phone.NUMBER));
                    contactsList.add(displayName + "\n" + number);
                }
                adapter.notifyDataSetChanged();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
            }
        }
    }
	
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions,
    	int[] grantResults) {
        switch (requestCode) {
            case 1:
                if (grantResult.length > 0 && grantResults[0] == PackageManager.
                   PERMISSION_GRANTED) {
                    readContacts();
                } else {
                    Toast.makeText(this, "You denied the permission", 
                                   Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }
}
```

最后在 AndroidManifest.xml 中声明

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.runtimepermissiontest">

    <uses-permission android:name="android.permission.READ_CONTACTS" />

    ...

</manifest>
```





## 创建自己的内容提供器



### 创建内容提供器的步骤

前面已经提到过，如果想要实现跨程序共享数据的功能，官方推荐的方式就是使用内容提供器，可以通过新建一个类去继承 ContentProvider 的方式来创建一个自己的内容提供器。ContentProvider 类中有 6 个抽象方法，我们在使用子类继承它的时候，需要将这 6 个方法全部重写。新建 MyProvider 继承自 ContentProvider，代码如下所示：

```java
public class MyProvider extends ContentProvider {
    @Override
    public boolean onCreate() {
        return false;
    }
    
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, 
                        String[] selectionArgs, String sortOrder) {
        returnnull;
    }
    
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        return null;
    }
    
    @Override
    public int update(Uri uri, ContentValues values, String selection,
                     String[] selectionArgs) {
        return 0;
    }
    
    @Override
    public int delete(Uri uri, String selection, 
                      String[] selectionArgs) {
        return 0;
    }
    
    @Override
    public String getType(Uri uri) {
        return null;
    }
}
```



> 1. onCreate()：初始化内容提供器的时候调用。通常会在这里完成对数据库的创建和升级等操作，返回 ture 表示内容提供器初始化成功，返回 false 则表示失败。注意，只有当存在 ContentResolver 尝试访问我们程序中的数据时，内容提供器才会被初始化。
> 2. query()：从内容提供器中查询数据。使用 uri 参数来确定查询哪张表，projection 参数用于确定查询那些列，selection 和 selectionArgs 参数用于约束查询那些行，sortOrder 参数用于对结果进行排序，查询的结果存放在 Cursor 对象中返回。
> 3. insert()：向内容提供器中添加一条数据。使用 uri 参数来确定要添加到的表，待添加的数据保存在 values 参数中。添加完成后，返回一个用于表示这条新记录的 URI。
> 4. updata()：更新内容提供器中已有的数据。使用 uri 参数来确定更新哪一张表中的数据，新数据保存在 values 参数中，selection 和 selectionArgs 参数用于约束更新那些行，受影响的行数作为返回值返回。
> 5. delete()：从内容提供器中删除数据。使用 uri 参数来确定删除哪一张表中的数据，selection 和 selectionArgs 参数用于约束删除那些行，被删除的行数将作为返回值返回。
> 6. getType()：根据传入的内容 URI 来返回响应的 MIME 类型。



对于 URI 写法还可以扩展，就是我们在要访问的表后面再加上一个 id，如下所示：

```
content://com.example.app.provider/table1/1
```

这就表示调用方期望访问的是 com.example.app 这个应用的 table1 表中 id 为 1 的数据。

内容 URI 的格式主要就只有以上两种，以路径结尾就表示期望访问该表中所有的数据，以 结尾就表示期望访问该表中拥有相应 id 的数据。我们可以使用通配符的方式来分别匹配这两种格式的内容 URI，规则如下。

> \* ：表示匹配任意长度的任意字符。
>
> \# ：表示匹配任意长度的数字。

所以，一个能够匹配任意表的内容 URI 格式就可以写成：

```
content://com.example.app.provider/*
```

所以，一个能够匹配 table1 表的任意一行数据的内容 URI 格式就可以写成：

```
content://com.example.app.provider/table1/#
```

接着，我们再借助 UriMatcher 这个类就可以轻松地实现匹配内容 URI 的功能。UIMatcher 中提供了一个 addURI() 方法，这个方法接收 3 个参数，可以分别把 authority 、path 和一个自定义代码传进去。这样当调用 URIMatcher 的 match() 方法时，就可以讲一个 Uri 对象传入，返回值是某个能够匹配这个 Uri 对象所对应的自定义代码，利用这个代码，我们就可以判断出调用方期望访问的是哪张表中的数据了。修改 MyProvider 中的代码，如下所示：

```java
public class MyProvider extends ContentProvider {
    
    public static final int TABLE1_DIR = 0;
    public static final int TABLE1_ITEM = 1;
    public static final int TABLE2_DIR = 2;
    public static final int TABLE2_ITEM = 3;
    public static UriMatcher uriMatcher;
    
    static {
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        uriMatcher.addURI("come.example.app.provider", "table1", TABLE1_DIR);
        uriMatcher.addURI("come.example.app.provider", "table1/#", TABLE1_ITEM);
        uriMatcher.addURI("come.example.app.provider", "table2", TABLE2_DIR);
        uriMatcher.addURI("come.example.app.provider", "table2/#", TABLE2_ITEM);
    }
    
    @Override
    public boolean onCreate() {
        return false;
    }
    
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, 
                        String[] selectionArgs, String sortOrder) {
        
        switch (uriMatcher.match(uri)) {
            case TABLE1_DIR:
                // 查询 table1 中的所有数据
                break;
            case TABLE1_ITEM:
                // 查询 table1 中的单条数据
                break;
             case TABLE2_DIR:
                // 查询 table2 中的所有数据
                break;
            case TABLE2_ITEM:
                // 查询 table2 中的单条数据
                break;
            default:
                break;
        }
        
        returnnull;
    }
    
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        return null;
    }
    
    @Override
    public int update(Uri uri, ContentValues values, String selection,
                     String[] selectionArgs) {
        return 0;
    }
    
    @Override
    public int delete(Uri uri, String selection, 
                      String[] selectionArgs) {
        return 0;
    }
    
    @Override
    public String getType(Uri uri) {
        return null;
    }
}
```

上述重写的 query() 方法，使用 `uriMatcher.match(uri)` 用来匹配 uri 内容，来判断期望访问的是什么数据。其实另外的几个方法也实现这个操作，只要携带 Uri 这个参数，就能够使用 UriMatcher 的 match() 方法判断出调用方期望访问的是哪张表。

除此之外还有一个方法 getType() 方法。它是所有的内容提供器都必须提供的一个方法，用于获取 Uri 对象所对应的 MIME 类型。一个内容 URI 所对应的 MIME 字符串主要由3部分组成，Android 对这 3 个部分做了如下格式规定。

* 必须以 vnd 开头。
* 如果内容 URI 以路径结尾，则后接 android.cursor.dir/，如果内容 URI 以 id 结尾，则后接 android.cursor.item/。
* 最后接上 vnd.\<authority\>.\<path\> 。

根据上述原则，可以继续完善 getType() 方法中的逻辑，代码如下：

```java
@Override
public String getType(Uri uri) {
    
    switch (uriMatcher.match(uri)) {
        case TABLE1_DIR:
            return "vnd.android.cursor.dir/vnd.come.example.app.provider.table1";
            
        case TABLE1_ITEM:
            return "vnd.android.cursor.item/vnd.come.example.app.provider.table1";
            
        case TABLE2_DIR:
            return "vnd.android.cursor.dir/vnd.come.example.app.provider.table2";
            
        case TABLE2_ITEM:
            return "vnd.android.cursor.item/vnd.come.example.app.provider.table2";
            
        default:
            break;
    }
    return null;
}
```

到这里，一个完整的内容提供器就创建完成了，现在任何一个应用程序都可以使用 ContentResolver 来访问我们程序中的数据。如何保证隐私数据不会泄漏出去？因为所有的 CRUD 操作都一定要匹配到响应的内容 URI 格式才能进行的，所以这部分数据根本无法被外部程序访问到，安全问题也就不存在了。





# 9 手机多媒体

【手机多媒体】通知、媒体调用



## 将程序运行到手机上

略

## 使用通知



### 通知的基本用法

通知的用法比较灵活，既可以在活动里创建，也可以在广播接收器里创建，相比于广播接收器和服务，在活动里创建通知的场景还是比较少的，一般只有当程序进入后台的时候我们才需要使用通知。不过，不论在哪里创建通知，整体的步骤都是相同的，下面我们就来学习一下创建通知的详细步骤。首先需要一个 NotificationManager 来对通知进行管理，可以调用 Context 的 getSystemService() 方法获取到。 getSystemService() 方法接收一个字符串参数用于确定获取系统的那个服务，这里我们传入 Context.NOTIFICATION_SERVICE 即可。因此获取 NotificationManager 的实例就可以写成：

```java
NotificationManager manager = 
    	(NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE);
```

接下来需要使用一个 Builder 构造器来创建 Notification 对象

```java
 Notification notification = new NotificationCompat.Builder(this)
                        .setContentTitle("this is content title")
                        .setContentText("This is content text")
                        .setWhen(System.currentTimeMillis())
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setContentIntent(pi)
                        .setPriority(NotificationCompat.PRIORITY_MAX)
                        .build();
```

setWhen() 方法用于指定通知被创建的时间，一毫秒为单位。

以上工作都完成后，只需调用 NotificationManager 的 notify() 方法就能让通知显示出来了。notify() 方法接收两个参数，一个参数是 id，要保证为每个通知所指定的 id 都是不同的，第二个参数则是 Notification 对象。以下未调用示例：

```java
manager.notify(1, notification);
```



下面新建一个 NotificationTest 项目，并修改 activity_main.xml 中的代码，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/send_notice"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Send notice" />

</LinearLayout>
```

添加一个按钮，用于发出通知。下面修改 MainActivity.java 中的代码，如下

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button sendNotice = (Button) findViewById(R.id.send_notice);
        sendNotice.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {

            case R.id.send_notice:
                NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
                Notification notification = new NotificationCompat.Builder(this)
                        .setContentTitle("this is content title")
                        .setContentText("This is content text")
                        .setWhen(System.currentTimeMillis())
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setPriority(NotificationCompat.PRIORITY_MAX)
                        .build();
                manager.notify(1, notification);
                break;
            default:
                break;
        }
    }
}

```

现在运行程序，点击按钮，就会看到系统状态栏中出现了一个通知。下面我们要为这个通知添加一个点击响应，这就涉及了一个新的概念：PendingIntent。

PendingIntent 于 Intent 有些相似，不同的是，Intent 更加倾向于去立即执行某个动作，而 PendingIntent 更加倾向于在某个合适的时机去执行某个动作。所以可以把PendingIntent 简单地理解为延时执行的 Intent。

PendingIntent 主要提供了几种静态调用方法用于获取 PendingIntent 的实例，根据需求可以选择使用 getActivity、getBroadcast、getService 这几种方法，这几种方法所接收的参数都是相同的，第一个是 Context。第二个参数一般用不到，通常传入 0 即可。第三个参数是一个 Intent 对象，我们可以通过这个对象构建出 PendingIntent 的意图。第四个参数用于确定 PendingIntent 的行为，有 FLAG_ONE_SHOT、FLAG_NO_CREATE、FLAG_CANCEL_CURRENT 和 FLAG_UPDATE_CURRENT 这 4 种值可选，通常情况下传入 0 就可以了。

NotificationCompat.Builder 这个构造器还可以再连缀一个 setContentIntent() 方法，接收的参数是一个 PendingIntent 对象。这里就可以通过 PendingIntent 对象构建出一个延迟执行的意图。

新建一个活动命名为 NotificationActivity 并添加一个布局命名为 notification_layout：

NotificationActivity.java

```java
public class NotificationActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.notification_layout);
        
        // 点击响应后自动清除通知
        NotificationManager manager = 
            (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        manager.cancel(1);
    }
}
```

notification_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".NotificationActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:textSize="24sp"
        android:text="This is notification layout"
        />

</RelativeLayout>
```

下面修改 MainActivity.java 代码

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button sendNotice = (Button) findViewById(R.id.send_notice);
        sendNotice.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {

            case R.id.send_notice:
                Intent intent = 
                    new Intent(this, NotificationActivity.class);
                PendingIntent pi = 
                    PendingIntent.getActivity(this, 0, intent, 0);
                NotificationManager manager = 
                    (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
                
                Notification notification = 
                    new NotificationCompat.Builder(this)
                        .setContentTitle("this is content title")
                        .setContentText("This is content text")
                        .setWhen(System.currentTimeMillis())
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setContentIntent(pi)
                        .setPriority(NotificationCompat.PRIORITY_MAX)
                        .build();
                manager.notify(1, notification);
                
                break;
            default:
                break;
        }
    }
}
```

点击响应清除通知还能在 NotificationCompat.Builder 这个构造器后面连缀一个 `setAutoCancel(true)` 来实现

```java
Notification notification = new NotificationCompat.Builder(this)
    ...
    .setAutoCancel(true)
    .build();
```



### 通知的进阶技巧

实际上 NotificationCompat.Builder 中提供了丰富的 API 能够创建出更加多样的通知效果，下面将介绍一些比较常用的 API。

* setSound() 方法 ：可以在通知发出的时候播放一段音频，接收一个 Uri 参数，在指定音频文件的时候还需要先获取到音频文件对应的 URI。例如：

```java
Notification notification = new NotificationCompat.Builder(this)
    ...
    .setSound(Uri.fromFile(
        new File("/system/media/audio/ringtones/Luna.ogg")))
    .build();
```

除了允许播放音频外，我们还可以在通知到来的时候让手机进行振动，使用的是 vibrate 这个属性。它是一个长整型的数组，用于设置手机静止和振动的时长，一毫秒为单位。下标为 0  的值表示手机静止的时长，下标为 1 的值表示手机振动的时长，下标为2的值又表示手机静止的时长，以此类推。例：让手机在通知到来的时候立刻振动1秒，然后静止1秒，再振动1秒：

```java
Notification notification = new NotificationCompat.Builder(this)
    ...
    .setVibrate(new long[]{ 0, 1000, 1000, 1000 })
    .build();
```

不过，想要控制手机振动还需要声明权限。因此，还需要编辑 AndroidManifest.xml 加入以下声明：

```xml
<uses-permission android:name="android.permission.VIBRATE" />
```



### 通知的高级功能

* setStyle() 方法 ：这个方法可以构建出富文本通知，也就是说通知中不光可以有文字和图标，还可以包含更多的东西。setStyle() 方法接收一个 NotificationCompat.Style 参数，这个参数就是用来构建具体的富文本信息的，如长文字、图片等。

**长文字**

在 setStyle() 方法中创建一个 NotificationCompat.BigTextStyle 对象，这个对象就是用于封账长文字信息的，我们调用它的 bigText() 方法并将文字传入就可以了。

```java
Notification notification = new NotificationCompat.Builder(this)
    ...
    .setStyle(new NotificationCompat.BigTextStyle()
             .bigText("...................................\
                      ....................................\
                      ....................................")) 
    .build();
```



除了显示长文字，通知里还可以显示一张大图片，具体用法也是基本相似：

```java
Notification notification = new NotificationCompat.Builder(this)
    ...
    .setStyle(new NotificationCompat.BigPictureStyle()
             .bigPicture(BitmapFactory.decodeResource(getResources(), 
             	R.drawable.big_image))) 
    .build();
```



* setPriority() 方法接收一个整型参数用于设置这条通知的重要程度，一共有5个常量值可选：

> PRIORITY_DEFAULT：表示默认重要程度，和不设置是一样的；
>
> PRIORITY_MIN：表示最低的重要程度，系统可能只会在特定场景才显示这条通知；
>
> PRIORITY_LOW：表示较低的重要程度，系统可能会将这类通知缩小，或改变显示顺序，将其排在更重要的通知之后；
>
> PRIORITY_HIGH：表示较高的重要程度，系统可能会将这类通知放大或改变其显示顺序，并将其排在比较靠前的位置；
>
> PRIORITY_MAX：表示最高的重要程度，这类通知必须要让用户立刻看到，甚至需要用户做出响应；

```java
Notification notification = 
    new NotificationCompat.Builder(this)
		...
        .setPriority(NotificationCompat.PRIORITY_MAX)
        .build();
```



## 调用摄像头和相册

新建一个项目 CameraAlbumTest ，然后修改 activity_main.xml 中的代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/take_photo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Take Photo" />

    <Button
        android:id="@+id/choose_from_album"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Choose From Album" />

    <ImageView
        android:id="@+id/picture"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal" />

</LinearLayout>
```

添加了两个 Button 和一个 ImageView。Button 是分别用于打开摄像头和相册的，ImageView 是用于将图片显示出来的。然后开始编写摄像头的具体逻辑，修改 MainActivity.java 中的代码。

```java
public class MainActivity extends AppCompatActivity {

    public static final int TAKE_PHOTO = 1;
    public static final int CHOOSE_PHOTO = 2;
    private ImageView picture;
    private Uri imageUri;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button takePhoto = (Button) findViewById(R.id.take_photo);
        Button chooseFromAlbum = (Button) findViewById(R.id.choose_from_album);
        picture = (ImageView) findViewById(R.id.picture);
        takePhoto.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 创建 File 对象，用于存储拍照后的图片
                File outputImage = 
                    new File(getExternalCacheDir(), "output_image.jpg");
                try {
                    if (outputImage.exists()) {
                        outputImage.delete();
                    }
                    outputImage.createNewFile();
                } catch (IOException e) {
                    e.printStackTrace();
                }

                imageUri = FileProvider.getUriForFile(
                    			MainActivity.this,
                    			"com.example.cameraalbumtest.fileprovider", 
                        		outputImage);
                
                // 启动相机程序
                Intent intent = 
                    new Intent("android.media.action.IMAGE_CAPTURE");
                
                intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                startActivityForResult(intent, TAKE_PHOTO);
            }
        });

        chooseFromAlbum.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (ContextCompat.checkSelfPermission(MainActivity.this,
                        Manifest.permission.WRITE_EXTERNAL_STORAGE) !=
                        PackageManager.PERMISSION_GRANTED) {
                    ActivityCompat.requestPermissions(MainActivity.this,
                            new String[]{ 
                                Manifest.permission.WRITE_EXTERNAL_STORAGE }, 1);
                } else {
                    openAlbum();
                }
            }
        });
    }

    private void openAlbum() {
        Intent intent = new Intent("android.intent.action.GET_CONTENT");
        intent.setType("image/*");
        startActivityForResult(intent, CHOOSE_PHOTO); // 打开相册
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, 
    	String[] permission, int[] grantResults) {
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && 
                    grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    openAlbum();
                } else {
                    Toast.makeText(this, "You denied the permission",
                            Toast.LENGTH_SHORT).show();
                }
                break;
            default:

        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {

        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case TAKE_PHOTO:
                if (resultCode == RESULT_OK) {
                    try {
                        // 将拍摄的照片显示出来
                        Bitmap bitmap = BitmapFactory.decodeStream(
                                getContentResolver().openInputStream(imageUri));

                        picture.setImageBitmap(bitmap);

                    } catch (FileNotFoundException e) {
                        e.printStackTrace();
                    }
                }
                break;
            case CHOOSE_PHOTO:
                if (resultCode == RESULT_OK) {
                    handleImageOmKitKat(data); // 处理图片
                }
            default:
                break;
        }
    }

    @TargetApi(19)
    private void handleImageOmKitKat(Intent data) {
        String imagePath = null;
        Uri uri = data.getData();
        if (DocumentsContract.isDocumentUri(this, uri)) {
            // 如果是 document 类型的 uri 则通过 document id 处理
            String docId = DocumentsContract.getDocumentId(uri);
            if("com.android.providers.media.documents".equals(uri.getAuthority())) {
                String id = docId.split(":")[1]; // 解析出数字格式的 id
                String selection = MediaStore.Images.Media._ID + "=" + id;
                imagePath = getImagePath(
                    MediaStore.Images.Media.EXTERNAL_CONTENT_URI, selection);
            } else if("com.android.providers.dowloads.documents".equals(
                			uri.getAuthority())) {
                Uri contentUri = ContentUris.withAppendedId( Uri.parse(
                    "content://downloads/public_downloads"), Long.valueOf(docId));
                
                imagePath = getImagePath(contentUri, null);
            }
        } else if("content".equalsIgnoreCase(uri.getScheme())) {
            // 如果是 content 类型的 uri 则使用普通方式处理
            imagePath = getImagePath(uri, null);
        } else if("file".equalsIgnoreCase(uri.getScheme())) {
            // 如果是 file 类型的 uri，直接获取图片路径即可
            imagePath = uri.getPath();
        }
        displayImage(imagePath);
    }

    private void displayImage(String imagePath) {
        if (imagePath != null) {
            Bitmap bitmap =  BitmapFactory.decodeFile(imagePath);
            picture.setImageBitmap(bitmap);
        } else {
            Toast.makeText(this, "failed to get image", Toast.LENGTH_SHORT).show();
        }
    }

    private String getImagePath(Uri uri, String selection) {
        String path = null;
        // 通过 Uri 和 selection 来获取真实的图片路径
        Cursor cursor = getContentResolver()
            				.query(uri, null, selection, null, null);
        if (cursor != null) {
            if(cursor.moveToFirst()) {
                path = cursor.getString(
                    cursor.getColumnIndex(MediaStore.Images.Media.DATA));
            }
            cursor.close();
        }
        return path;
    }


}
```

上述代码中调用相机部分，首先创建了一个 File 对象用于存放摄像头拍下的照片，我们把图片命名为output_image.jpg ，并将它存放在手机应用相关联的缓存目录下。调用 getExternalCacheDir() 方法可以得到这个目录，接着调用getUrifroFile() 方法接收三个参数，第一个参数要求是 Context 对象，第二个参数可以是任意唯一的字符串，第三个参数是刚刚创建的 File 对象。

接下来构建出一个 Intent 对象，并将这个 Intent 的 action 指定为 android.media.action.IMAGE_CAPTURE在，再调用 Intent 的 putExtra() 方法指定图片的输出地址，这里填入刚刚得到的 Uri 对象，最后调用 startActivityForResult() 来启动活动。由于我们使用的是一个隐式 Intent， 系统会找出能够响应这个 Intent 的活动去启动，拍下来的照片将会输出到 output_image.jpg 中。

由于 startActivityForResult() 是会回调 onActivityResult() 的，因此可以添加一条分支，如果发现拍照成功，就可以调用 BitmapFactory 的 decodeStream() 方法将 output_image.jpg 这张照片解析成 Bitmap 对象，然后把它设置到 ImageView 中显示出来。

对于图库的使用，我们需要动态申请 WRITE_EXTERNAL_STORAGE 这个危险权限，因为需要从 SD 卡中读取照片就需要申请这个权限。WRITE_EXTERNAL_STORAGE 表示同时授权程序对 SD 卡读和写的能力。当用户授权了权限申请之后会调用 openAlbum() 方法，这里先是构建了一个 Intent 对象，并将它的 action 指定为 android.intent.action.GET_CONTENT。接着给这个 Intent 对象设置一些必要的参数，然后调用 startActivityForResult() ，这样我们在 onActivityResult() 中添加图库调用的分支，如果读取图片成功，就执行 handleImageOmKitKat() 方法，这个方法中的逻辑基本内容是解析这个封装过的 Uri 。如果返回的 Uri 是 document 类型的话，那就取出 document id 进行处理，如果不是的话，那就使用普通的方式处理。另外，如果的 authority 是 media 格式的话，document id 还需要再进行一次解析，要通过字符串分割的方式取出后半部分才能得到真正的数字 id。取出的 id 用于构建新的 Uri 和条件语句，把这些值作为参数传入到 getImagePath() 方法中就可以获取到图片的真实路径了。拿到图片的路径之后，再调用 displayImage() 方法将图片显示到界面上。

接下来需要在 AndroidManifest.xml 中注册相关权限和内容提供器

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.cameraalbumtest">
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.CameraAlbumTest">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <provider
            android:authorities="com.example.cameraalbumtest.fileprovider"
            android:name="androidx.core.content.FileProvider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />

        </provider>
    </application>

</manifest>
```

其中，android:name 属性值是固定的， android:authorities 属性值必须要和刚才 FileProvider.getURIForFile() 方法中的第二个参数一致。另外在 `<provider>` 标签的内部使用 `<meta-data>` 来指定 Uri 的共享路径，并引用了一个 @xml/file_paths 资源。当然，这个资源现在是不存在的，马上来创建它

/res/xml/file_paths.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="" />
</paths>
```

其中，external-path 就是用来指定 Uri 共享的，name 属性可以随意填，path 属性的值表示共享的具体路径。这设置为空值就是表示将整个 SD 卡进行共享，当然你也可以仅共享我们存放 output_image.jpg 这张图片的路径。

运行程序，点击两个按钮测试，均可正常操作。



## 播放多媒体文件

### 播放音频

在 Android 中音频文件一般都是使用 MediaPlayer 类来实现的，它对多种格式的音频文件提供了非常全面的控制方法，从而使得播放音乐的工作变动十分简单。下表列出了 MediaPlayer 类中一些较为常用的控制方法。

| 方法名          | 功能描述                                                    |
| --------------- | ----------------------------------------------------------- |
| setDataSource() | 设置要播放的音频文件的位置                                  |
| prepare()       | 在开始播放之前调用这个方法完成准备工作                      |
| start()         | 开始或继续播放音频                                          |
| pause()         | 暂停播放音频                                                |
| reset()         | 将MediaPlayer对象重置到刚刚创建的状态                       |
| seekTo()        | 从指定的位置开始播放音频                                    |
| stop()          | 停止播放音频。调用这个方法后的MediaPlayer对象无法在播放音频 |
| release()       | 释放掉与MediaPlayer对象相关的资源                           |
| isPlaying()     | 判断当前MediaPlayer是否正在播放音频                         |
| getDuration()   | 获取载入的音频文件的时长                                    |

MediaPlayer 的工作流程。首先需要创建一个 MediaPlayer 对象，然后调用 setDataSource() 方法来设置音频文件的路径，再调用 pause() 方法就会暂停播放，调用 reset() 就会停止播放。

新建一个 PlayAudioTest 项目，然后修改 activity_main.xml 中的代码，如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/play"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Play" />

    <Button
        android:id="@+id/pause"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Pause" />

    <Button
        android:id="@+id/stop"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stop" />

</LinearLayout>
```

然后修改 MainActivity.java

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private MediaPlayer mediaPlayer = new MediaPlayer();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button play = (Button) findViewById(R.id.play);
        Button pause = (Button) findViewById(R.id.pause);
        Button stop = (Button) findViewById(R.id.stop);

        play.setOnClickListener(this);
        pause.setOnClickListener(this);
        stop.setOnClickListener(this);
        
        if(ContextCompat.checkSelfPermission(MainActivity.this, 
                Manifest.permission.WRITE_EXTERNAL_STORAGE) !=
                PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(MainActivity.this, 
                    new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
        } else {
            initMediaPlayer(); // 初始化 MediaPlayer 
        }
        

    }

    private void initMediaPlayer() {
        try {
            File file = new File(Environment.getDownloadCacheDirectory(),
                    "music.mp3");
            mediaPlayer.setDataSource(file.getPath()); // 指定音频文件的路径
            mediaPlayer.prepare(); // 让 MediaPlayer 进入到准备状态
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions,
                                           int[] grantResults) {
        switch (requestCode) {
            case 1:
                if(grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    initMediaPlayer();
                } else {
                    Toast.makeText(this, "拒绝权限无法使用",
                            Toast.LENGTH_SHORT).show();
                    finish();
                }
                break;
            default:
        }
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.play:
                if(!mediaPlayer.isPlaying()) {
                    mediaPlayer.start();
                }
                break;
            case R.id.pause:
                if(!mediaPlayer.isPlaying()) {
                    mediaPlayer.pause();
                }
                break;
            case R.id.stop:
                if(!mediaPlayer.isPlaying()) {
                    mediaPlayer.stop();
                }
                break;
            default:
                break;
        }
    }


    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(mediaPlayer != null) {
            mediaPlayer.stop();
            mediaPlayer.release();
        }
    }


}
```

最后在 AndroidManifest.xml 中添加权限

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.playaudiotest">
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	...
</manifest>
```



### 播放视频

播放视频与播放音频类似，主要是使用 VideoView 类来实现的。这个类将视频的显示和控制集于一身，使得我们仅仅借助它就可以完成一个建议的视频播放器。主要用法如下：

| 方法名         | 功能描述                   |
| -------------- | -------------------------- |
| setVideoPath() | 设置要播放的视频文件的位置 |
| start()        | 开始或继续播放视频         |
| pause()        | 暂停播放视频               |
| seekTo()       | 从指定的位置开始播放视频   |
| isPlaying()    | 判断当前是否正在播放视频   |
| getDuration()  | 获取载入的视频文件的时长   |
| suspend()      | 释放资源                   |

新建项目 PlayVideoTest 项目，然后修改 activity_main 中的代码

```xml

```

接下修改 MainActivity.java 代码

```java

```

最后在 AndroidManif.xml 添加权限

```xml

```

最后运行程序，符合预期。



# 10 网络技术

【网络技术】WebView、网络请求、格式解析





## WebView 的用法

Android 支持开发者在应用内部嵌入一个浏览器，可以通过 WebView 控件来实现，新建 WebView 项目。

修改 activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <WebView
        android:id="@+id/web_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    
</LinearLayout>
```

然后修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		// 获取实例
        WebView webView = (WebView) findViewById(R.id.web_view);
        // 支持JavaScript脚本
        webView.getSettings().setJavaScriptEnabled(true);
        //当需要一个网页跳转到另一个网页时 目标网页仍然在当前网页显示
        webView.setWebViewClient(new WebViewClient());
        webView.loadUrl("http://www.baidu.com");
    }
}
```

最后注册权限

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.webviewtest">
    <uses-permission android:name="android.permission.INTERACT_ACROSS_PROFILES"/>
	...
</manifest>
```



## 使用 HTTP 协议访问网络

使用手动发送 HTTP 请求的方式，深入地理解这一个过程。



### 使用 HttpURLConnection

在 Android 上发送 HTTP 请求一般都使用 HttpURLConnection。首先需要获取到 HttpURLConnection 的实例，一般只需要 new 一个对象 URL 对象出来，并传入目标网络地址，然后调用一下openConnection() 方法即可，如图所示：

```java
URL url = new URL("http://www.baidu.com");
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
```

在得到了 HttpURLConnection 的实例之后，我们可以设置一下 Http 请求所使用的的方法。常用的方法主要有两个：GET 和 POST。 GET 表示希望从服务器那里获取数据，而 POST 表示希望提交数据给服务器，写法如下：

```java
connection.setRequestMethod("GET");
```

接下来就可以进行一些自由地定制了，比如设置连接超时、读取超时的毫秒数，以及服务器希望得到的一些消息头等。例：

```java
connection.setConnectTimeout(8000);
connection.setReadTimeout(8000);
```

之后再调用 getInputStream() 方法就可以获取到服务器返回的输入流了，剩下的任务就是对输入流进行读取，如下所示：

```java
InputStream in = connection.getInputStream();
```

最后调用 disconnect() 方法将这个 HTTP 连接关闭掉，如下所示

```java
connection.disconnect();
```



新建一个 NetworkTest 项目，首先修改 activity_main.xml 中的代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/send_request"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send request" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
        <TextView
            android:id="@+id/response_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </ScrollView>
    
</LinearLayout>
```

这里使用了一个新控件：ScrollView，如果显示的内容过多，可以借助 ScrollView 控件的话，我们就可以以滚动的形式查看屏幕外的那部分内容。布局中还放置了一个 Button 和一个 TextView， 用于请求响应和显示数据。

接着修改 MainActivity.java 中的代码：

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    TextView responseTest;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button sendRequest = (Button) findViewById(R.id.send_request);
        responseTest = (TextView) findViewById(R.id.response_text);
        sendRequest.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        if(v.getId() == R.id.send_request) {
            sendRequestWithHttpURLConnection();
        }
    }

    private void sendRequestWithHttpURLConnection() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try {
                    URL url = new URL("http://www.baidu.com");
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(8000);
                    connection.setReadTimeout(8000);
                    InputStream in = connection.getInputStream();

                    // 下面对获取到的输入流进行读取
                    reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while((line = reader.readLine()) != null) {
                        response.append(line);
                    }
                    showResponse(response.toString());
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    private void showResponse(final String response) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // 这里进行 UI 操作，将结果显示到界面上
                responseTest.setText(response);
            }
        });
    }
}
```

可以看到，我们在 Send Request 按钮的点击事件里调用了  方法，在这个方法中显示开启了一个子线程，然后在子线程里使用 HttpURLConnection 发出一条 HTTP 请求，请求的目标地址就是百度的首页。接着利用 BufferedReader 对服务器返回的流进行读取，并将结果传入到了  方法中。而在  方法里则是调用了一个  方法，然后在这个方法的匿名类参数中进行操作，将返回的数据显示到界面上。因为 Android 是不允许在子线程中进行 UI 操作的，所以需要通过  方法将线程写环岛主线程，然后再更新 UI 元素。

最后声明权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

运行程序点击按钮，就能看到返回的 html 文件的代码了。

如果是提交数据给服务器，只需要将请求方法改成 POST，并在获取输入流之前把要提交的数据写出即可。每条数据都要以键值对的形式存在，数据与数据之间用 “&” 隔开，例：

```java
connection.setRequestMethod("POST");
DataOutputStream out = 
    new DataOutputStream(connection.getOutputStream());
out.writeBytes("username=admin&password=123456");
```





### 使用 OkHttp

OkHttp 是 Square 公司开发的，OKHttp 不仅在接口上封装上边做得很简单易用之外，就连底层实现也是自成一派，比起原生的 HttpURLConnect，可以说是有过之而无不及，现在已经成可广大 Android 开发者首选的网络通信库。OkHttp 的项目的主页网址是：https://github.com/square/okhttp 。

在使用 OkHttp 之前，需要添加 OkHttp 库的依赖。编辑 app/build.gradle 文件，在 dependencies 闭包中添加如下内容

```groovy
dependencies {
	compile fileTree(dir: 'libs', iclude: ['*.jar'])
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
    compile 'com.squareup.okhttp3:okhttp:3.4.1'
}
```

添加上述依赖会自动下载两个库，一个是 OKHttp 库，一个是 Okio 库，后者是前者的通信基础。3.4.1 为版本号，根据时间和需求自己修改。

* OKHttp 的具体用法

首先创建一个 OkHttpClient 的实例，如下所示：

```java
okHttpClient client = new OkHttpClient();
```

接下来如果想要发起一条 HTTP 请求，就需要创建一个 Request 对象：

```java
Request request = new Request.Builder().build();
```

当然，上述代码只是创建了一个空的 Request 对象，并没有实际作用，我们可以在最终的 build() 方法之前连缀很多其他方法来丰富这个Request 对象。比如可以通过 url() 方法来设置目标网络地址，如下所示：

```java
Request request = new Request.Builder()
    	.url("http://www.baidu.com")
    	.build();
```

之后调用 OkHttpClient 的 newCall() 方法来创建一个 Call 对象，并调用它的 execute() 方法来发送请求并获取服务器返回的数据，写法如下：

```java
Response response = client.newCall(request).execute();
```

其中 Response 对象就是服务器返回的数据了，我们可以使用如下写法来得到返回的具体内容：

```java
String responseData = response.body().string();
```

如果是发起一条 POST 请求会比 GET 请求稍微复杂一点，我们需要先构建出一个 Request Body 对象来存放待提交的参数，如下：

```java
RequestBody requestBoday = new FormBody.Builder()
    	.add("username", "admin")
    	.add("password", "123456")
    	.build();
```

然后在 Request.Builder 中调用一下 post() 方法，并将 RequestBody 对象传入：

```java
Request request = new Request.Builder()
    	.url("http://www.baidu.com")
    	.post(requestBody)
    	.build();
```

接下来的操作就和 GET 请求一样可，调用 execute() 方法来发送请求并获取服务器返回的数据即可。





## 解析 XML 格式数据

网络上传输的数据一般都是一些格式化的数据，最常用的主要有两种，XML 和 JSON。

### 准备 xml 文件

搭建一个简单的 web 服务器，这里推荐 Apache 。开启服务，并在 Apache\htdocs 目录下，在这里新建一名为 get_data.xml 的文件，然后编辑这个文件，并加入下 XML 格式的内容。

```xml
<apps>
	<app>
    	<id>1</id>
        <name>Google Maps</name>
        <version>2.1</version>
    </app>
    <app>
    	<id>2</id>
        <name>Chrome</name>
        <version>1.0</version>
    </app>
    <app>
    	<id>3</id>
        <name>Google Play</name>
        <version>2.3</version>
    </app>
</apps>
```

这时就可以在浏览器里访问 http://127.0.0.1/get_data.xml 这个网址，就能看见文件的内容。

### Pull 解析方式

继续接续 NetworkTest 项目的基础上继续修改，修改 MainActivity.java 中的代码，如下所示：

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    TextView responseTest;
	...
    @Override
    public void onClick(View v) {
        if(v.getId() == R.id.send_request) {
            sendRequestWithOkHttp();
        }
    }

    private void sendRequestWithOkHttp() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try {
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                        	.url("http://10.0.2.2/get_data.xml")
                        	.build();
                    Response response = client.newCall(request).execute();
                    String responseData = response.body().string();
                    ParseXMLWithPull(responseData);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

  	...
    private void parseXMLWithPull(String xmlData) {
        try {
            XmlPullParserFactory factory = 
                XmlPullParserFactory.newInstance();
            XmlPullParser xmlPullParser = factory.newPullParser();
            XmlPullParser.setInput(new SringReader(xmlData));
            int eventType = xmlPullParser.getEventType();
            String id = "";
            String name = "";
            String version = "";
            while (eventType != XmlPullParser.END_DOCUMENT) {
                String nodeName = xmlPullParser.getName();
                switch(eventType) {
                        // 开始解析某个节点
                    case XmPullParser.START_TAG: {
                        if("id".equals(nodeName)) {
                            id = XmPullParser.nextText();
                        } else if ("name".equals(nodeName)) {
                            name = XmPullParser.nextText();
                        } else if ("version".equals(nodeName)) {
                            version = XmPullParser.nextText();
                        }
                        break; 
                    }
                    // 完成解析某个节点
                    case XmPullParser.END_TAG: {
                        if("app".equals(nodeName)) {
                            Log.d("MainActivity", "id is" + id);
                            Log.d("MainActivity", "name is" + name);
                            Log.d("MainActivity", "version is" + version);
                        }
                        break;
                    }
                    default:
                        break;
                }
                eventType = xmlPullParser.next();
        	}
    	} catch (Exception e) {
    		e.printStackTrace();        
    	}
    }
}
```

首先要获取到一个 XmlPullParserFactory 的实例，借助这个实例得到 XmlPullParser 对象，然后调用 XmlPullParser 的 setInput() 方法将服务器返回的 XML 数据设置进去就可以开始解析了。解析过程也非常简单，通过 getEventType() 可以得到当前的解析事件，然后在一个 while 循环中不断地进行解析，如果当前解析事件不等于 XmlPullParser.END_DOCUMENT，说明解析工作还没完成，调用 next() 方法后可以获取一个解析事件。

在 while 循环中，我们通过 getName() 方法得到当前节点的名字，如果发现节点名等于 id、name 或 version，就调用 nextText() 方法来获取节点的内容，每当解析完一个app节点后旧件获取到的内容打印出来。



### SAX 解析方式

Pull 解析方式虽然非常好用，但它并不是唯一的选择。SAX 解析也是一种特别常用的 XML 解析方式，虽然它的用法比 Pull 解析要复杂一些，但在语义方面会更加清楚。

通常情况下我们都会新建一个类继承自DefaultHandler，并重写父类的5个方法，如下：

```java
public class MyHandler extends defaultHandler{
    
    @Override
    public void startDocument() throw SAXException {        
    }
    
    @Override void startElement(String uri, String localName, String qName, 
                                Attributes attributes) throw SAXException {        
    }
    
    @Override
    public void characters(char[] ch, int start, 
                           int length) throw SAXException {        
    }
    
    @Override
    public void endElement(String uri, String localName, String qNmae) 
        throw SAXException {       
    }
    
    @Override
    public void endDocument() throw SAXException {   
    }
    
}
```

startDocument() 会在开始 XML 解析的时候调用，startElement() 会在开始解析某个节点的时候调用，characters() 方会在获取节点中内容的时候调用，endElement() 会在完全完成解析某个节点的时候调用，endDocument() 会在完成整个 XML 解析的时候调用。

下面尝试使用 SAX 解析的方式来实现和上一个小结同样地功能。新建一个 ContentHandler 类继承自 DefaultHandler，并重写父类的5个方法。

```java
public class ContentHandler extends defaultHandler{
    
    private String nodeName;
    private StringBuilder id;
    private StringBuilder name;
    private StringBuilder version;
    
    @Override
    public void startDocument() throw SAXException {  
        id = new StringBuilder();
        name = new StringBuilder();
        version = new StringBuilder();
    }
    
    @Override void startElement(String uri, String localName, String qName, 
                                Attributes attributes) throw SAXException {
        // 记录当前节点名
        nodeName = localname;
    }
    
    @Override
    public void characters(char[] ch, int start, 
                           int length) throw SAXException {
        // 根据当前的节点名判断将内容添加到哪一个 StringBuilder 对象中
        if("id".equals(nodeName)) {
            id.append(ch, Start, length);
        } else if("name".equals(nodeName)) {
            name.append(ch, start, length);
        } else if("version".equals(nodeName)) {
            version.append(ch, start. length);
        }
    }
    
    @Override
    public void endElement(String uri, String localName, String qNmae) 
        throw SAXException {
        if("app".equals(localName)) {
            Log.d("ContentHandler", "id is " + id.toString().trim());
            Log.d("ContentHandler", "name is " + name.toString().trim());
            Log.d("ContentHandler", "version is " + version.toString().trim());
            // 最后要将 StringBuilder 清空掉
            id.setLength(0);
            name.setLength(0);
            version.setLength(0);
        }
    }
    
    @Override
    public void endDocument() throw SAXException {
        super.endDocument();
    }
    
}
```

上述代码首先给 id、name 和 version 节点分别定义了一个 StringBuilder 对象，并在 startDocument() 方法里对它们进行了初始化。每当开始解析摸个节点的时候， startElement() 方法就会得到调用，其中 localName参数记录这当前节点的名字，这里我们把它记录下来。接着在解析节点中具体内容的时候就会调用 characters() 方法，我们会根据当前的节点名进行判断，将解析出的内容添加到哪一个 StringBuilder 对象中。最后在 endElement() 方法中进行判断，如果 app 节点已经解析完成，就将解析的内容打印出来，在打印之前还需要调用一下 trim() 方法，并且打印完成后还要将 StringBuilder 的内容清空掉。

接下来修改 MainActivity.java 

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    TextView responseTest;
	...

    private void sendRequestWithOkHttp() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {

                try {
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                        	.url("http://10.0.2.2/get_data.xml")
                        	.build();
                    Response response = client.newCall(request).execute();
                    String responseData = response.body().string();
                    ParseXMLWithSAX(responseData);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

  	...
    private void parseXMLWithSAX(String xmlData) {
        try {
            SAXParserFactory factory = SAXParserFactory.newInstance();
            XMLReader xmlReader = factory.newSAXParser().getXMLReader();
			ContentHandler handler = new ContentHandler();
            // 将 ContentHandler 的实例设置到 XMLReader 中
            xmlReader.setContentHandler(handler);
            // 开始执行解析
            xmlReader.parse(new InputSource(new StringReader(xmlData)));
    	} catch (Exception e) {
    		e.printStackTrace();        
    	}
    }
}
```

得到服务器返回的数据后，这次去调用 parseXMLWithSAX() 方法来解析 XML 数据。parseXMLWithSAX() 方法中先是创建了一个 SAXParserFactory的对象，然后在获取到 XMLReader 对象，接着我们将编写的 ContentHandler 的实例设置到 XMLReader 中，最后调用 puarse() 方法开始执行就好了。

**除了 Pull 和 SAX 之外，还有一种 DOM 解析方式也算挺常用的。**



## 解析 JSON 格式数据

### 准备 JSON 文件

JSON 文件相比于 XML 主要优势在于 JSON 的体积更小，在网络上传时可以更省流量。但缺点在于，它的语义性较差，看起来不如 XML 直观。

于 XML 相似，需要在 Apache\htdocs 目录下，在这里新建一名为 get_data.json 的文件，然后编辑这个文件，并加入下 JSON 格式的内容。

```json
[{"id":"5","version":"5.5","name":"Clash of Clans"},
{"id":"6","version":"7.0","name":"Boom Beach"},
{"id":"7","version":"3.5","name":"Clash Royale"}]
```



### 使用 JSONObject

修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

	...

    private void sendRequestWithOkHttp() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                        	.url("http://10.0.2.2/get_data.xml")
                        	.build();
                    Response response = client.newCall(request).execute();
                    String responseData = response.body().string();
                    ParseJSONWithJSONObject(responseData);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
  	...
    private void ParseJSONWithJSONObject(String josnData) {
        try {
            JSONArray jsonArray = new JSONArray(jsonData);
            for(int i = 0; i < jsonArray.length(); i++){
                JSONObject jsonObject.getJSONObject(i);
                String id = jsonObject.getString("id");
                String name = jsonObject.getString("name");
                String version = jsonObject.getString("version");
                Log.d("MainActivity", "id is" + id);
                Log.d("MainActivity", "name is" + name);
                Log.d("MainActivity", "version is" + version);               
            }
    	} catch (Exception e) {
    		e.printStackTrace();        
   		}
    }
}
```





### 使用 GSON

GSON 并没有被添加到 Android 官方的 API 中，因此如果想要这个功能的话，必须要在项目中添加 GSON 库的依赖。编辑 app/build.gradle 文件，在 dependencies 闭包中添加如下内容

```groovy
dependencies {
	compile fileTree(dir: 'libs', iclude: ['*.jar'])
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
    compile 'com.squareup.okhttp3:okhttp:3.4.1'
    compile 'com.google.code.gson:gson:2.7'
}
```

GSON 的工作原理是可以将一段 JSON 格式的字符串自动映射成一个对象，从而不需要再手动编写代码进行解析了。

比如说一段 JSON 格式的数据如下所示：

```json
{"name":"Tom","age":20}
```

那么就可以定义一个 Person 类，并加入 name 和 age 这两个字段，然后只需要简单地调用如下代码就可以将JSON数据自动解析成一个 Person 对象了：

```java
Gson gson = new Gson();
Person person = gson.fromJson(jsonData, Person.class);
```

如果是一段 JSON 数组会稍微麻烦一点，需要借助 TypeToken 将期望解析成的数据类型传入到 fromJson() 方法中，如下所示

```java
List<Person>people = gson.fromJson(JsonData, 
		new TyprToken<List<Person>>(){}.getType());
```



示例：新增一个 APP 类，并加入 id、name 和 version 这 3 个字段

```java
public class App {
    private String id;
    private String name;
    private String version;
    
    public String getId() {
        return id;
    }
    
    public void setId(String id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getVersion() {
        return version;
    }
    
    public void setVersion(String version) {
        this.versionversion = version;
    }
}
```

然后修改 MainActivity.java 中的代码

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

	...

    private void sendRequestWithOkHttp() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                        	.url("http://10.0.2.2/get_data.xml")
                        	.build();
                    Response response = client.newCall(request).execute();
                    String responseData = response.body().string();
                    ParseJSONWithGSON(responseData);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
  	...
    private void ParseJSONWithGSON(String josnData) {
        Gson gson = new Gson();
        List<App> appList = gson.fromJson(
            	jsonData, new TypeToken<List<App>>(){}.getType());
        
        for(App app : appList){
            JSONObject jsonObject.getJSONObject(i);
            String id = jsonObject.getString("id");
            String name = jsonObject.getString("name");
            String version = jsonObject.getString("version");
            Log.d("MainActivity", "id is" + id);
            Log.d("MainActivity", "name is" + name);
            Log.d("MainActivity", "version is" + version);               
        }
    }
}
```

以上就是两种格式的数据解析方式。



## 网络编程的最佳实践

为了方便调用和减少代码量，通常情况下会将这些通用的网络操作提取到一个公共类里，并提供一个静态方法，当想要发起网络请求的时候，只需要简单地调用一下这个方法即可。比如使用如下的写法：

```java
public class HttpUtil {

    public static String sendHttpRequest(String address) {
        HttpURLConnection connection = null;
        try {
            URL url = new URL(address);
            connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.setReadTimeout(8000);
            connection.setConnectTimeout(8000);
            connection.setDoInput(true);
            connection.setDoOutput(true);
            InputStream in = connection.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(in));
            StringBuilder response = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                response.append(line);
            }
            
            return response.toString();
            
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        } finally {
            if(connection != null) {
                connection.disconnect();
            }
        }
    }
}

```

以后每当需要发起一条 HTTP 请求时就可以这样写：

```java
String address = "http://www.baidu.com";
String response = HttpUtil.sendHttpRequest(address);
```

在获取到服务器响应的数据后，我们就可以对它进行解析和处理了。但是需要注意，网络请求通常都是属于耗时操作，而 sendHttpRequest() 方法的内部并没有开启线程，这样就有可能导致再调用 sendHttpRequest() 方法的时候使得主线程被阻塞住。但是在 sendHttpRequest() 方法内部开启一个线程来发起 HTTP 请求，那么服务器响应的数据是无法返回的，所有的耗时逻辑都是在子线程里进行的，sendHttpRequest() 方法会在服务器还没来得及响应的时候就执行结束了，当然也就无法返回响应的数据了。

目前一个比较好的解决方法就是使用 Java 的回调机制。首先需要定义一个接口，比如将它命名成为 HttpCallbackListener 代码如下：

```java
public interface HttpCallbackListener {
    void onFinish(String response);
    void onError(Exception e);
}
```

可以看到，我们在接口中定义了两个方法，onFinish() 方法表示当服务器成功响应我们请求的时候调用，onError() 表示当进行网络操作出现错误的时候调用。这两个方法都带有参数，onFinish() 方法中的参数代表着服务器返回的数据，而 onError() 方法中的参数记录着错误的详细信息。

接着修改 HttpUtil 中的代码，如下所示：

```java
public class HttpUtil {

    public static String sendHttpRequest(final String address, 
    						final HttpCallbackListener listener) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    URL url = new URL(address);
                    connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setReadTimeout(8000);
                    connection.setConnectTimeout(8000);
                    connection.setDoInput(true);
                    connection.setDoOutput(true);
                    InputStream in = connection.getInputStream();
                    BufferedReader reader = 
                        new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
            		while ((line = reader.readLine()) != null) {
                		response.append(line);
            		}
                    if(listener != null) {
                        // 回调 onFinish() 方法
                        listener.onFinish(response.toSring());
                    }
                } catch (Exception e) {
                    if (listener != null) {
                        // 回调 onError() 方法
                        listener.onError(e);
                    }
                } finally {
                    if(connection != null) {
                        conncetion.disconnect();
                    }
                }
            }
        }).start();
        
    }
}

```

首先给 sendHttpRequest() 方法添加了一个 HttpCallbackListener 参数，并在方法的内部开启了一个子线程，然后子线程里去执行具体的网络操作。注意，子线程中是无法通过 return 语句来返回数据的，因此我们将服务器响应的数据传入了 HttpCallbackListener 的 onFinish() 方法中，如果出现了异常就将异常的原因传入到 onError() 方法中。现在 sendHttpRequest() 方法接收两个参数了，因此我们在调用它的时候还需要将HttpCallbackListener 的实例传入，如下：

```java
HttpUtil.sendHttpRequest(address, new HttpCallbackListener() {
    @Override
    public onFinish(String response) {
        // 添加返回内容的执行逻辑
    } 
    @Override
    public void onError(Exception e){
        // 对异常情况进行处理
    }
});
```



# 11 服务

【服务】

## 服务简介

服务（Service）是 Android 中实现程序后台运行的解决方案，它非常合适去执行那些不需要和用户交互而且要求长期运行的任务。服务的运行不依赖于任何用户界面，即使程序被切换到后台，或者用户打开了另外一个应用程序，服务仍然能够保持正常运行。

服务并不是运行在一个独立的进程当中的，而是依赖于创建服务时所在的应用程序进程。当某个应用程序的进程被杀掉时，所有依赖于该进程的服务也会停止运行。实际上服务并不会自动开启线程，所有代码都是默认运行在主线程当中的。也就是说，我们需要在服务内部手动创建子线程，并在这哭执行具体的任务，否则就是有可能出现主线程被阻塞住的情况。



## Android 多线程编程



### 线程的基本用法

Android 多线程编程其实并不比 Java 多线程编程特殊，基本都是使用相同的语法，比如说，定义一个线程只需要新建一个类继承自 Thread，然后重写父类的 run() 方法。并在里面编写耗时逻辑即可，如下所示：

```java
class MyThread extends Thread {
    
    @Override
    public void run() {
        // 处理具体逻辑
    }
}
```

只需要 new 出 MyThread 的实例，然后调用它的 start() 方法，这样 run() 方法中的代码就会在子线程当中运行了，如下：

```java
new MyThread().start();
```

当然，使用继承的方式耦合性有点高，更多的时候我们都会选择使用实现 Runnable 接口的方式来定义一个线程，如下：

```java
class MyThread implements Runnable {
    
    @Override
    public void run() {
        // 处理具体逻辑
    }
}
```

如果使用了这种写法，启动线程的方法也需要进行相应的改变，如下所示：

```java
MyThread myThread = new MyThread();
new Thread(myThread).start();
```

可以看到，Thread 的构造函数接收一个 Runnable 参数，而我们 new 出的 MyThread 的实例，实现了 Runnable 接口的对象，所以可以直接将它传入到 Thread 的构造函数里。然后调用它的 start() 方法，这样 run() 方法中的代码就会在子线程当中运行了。

当然，如果你不想专门再定义一个类去实现 Runnable 接口，也是可以使用匿名类的方式，这种写法更为常见，如下

```java
new Thread(new Runnable(){
    @Override
    public void run(){
        // 处理具体的逻辑
    }
}).start();
```



### 在子线程中更新 UI

和许多其它的 GUI 库一样，Android 的 UI 也是线程不安全的。也就是说，如果想要更新应用程序里的 UI 元素，则必须在主线程中进行，否则就会出现异常。

新建一个 AndroidThreadTest 项目，然后修改 activity_main.xml 中的代码，如下所示

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/change_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Text" />

    <TextView
        android:id="@+id/text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:layout_centerInParent="true"
        android:textSize="20sp" />

</RelativeLayout>
```

接下来修改 MainActivity.java 中的代码，如下所示

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private TextView text;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        text = (TextView) findViewById(R.id.text);
        Button changeText = (Button)findViewById(R.id.change_text);
        changeText.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.change_text:
                
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        text.setText("Nice to meet you!");
                    }
                }).start();
                break;
            default:
                break;
        }
    }
}
```

可以看到，我们在 Change Text 按钮的点击事件里面开启了一个子线程，然后在子线程中调用 TextView 的 setText() 方法将显示的字符串改成 Nice to meet you。且是在子线程中更新 UI 的。运行一下程序，点击 按钮，就会发现程序果然崩溃了。



对于这种情况，Android 提供了一套异步消息处理机制，完美地解决了在子线程中进行 UI 操作的问题。修改 MainActivity.java 中的代码：

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    public static final int UPDATE_TEXT = 1;
    private TextView text;
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case UPDATE_TEXT:
                    // 这里可以进行 UI 操作
                    text.setText("Nice to meet you");
                    break;
                default:
                    break;
            }
        }
    };
	... ... ...

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.change_text:
                
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Message message = new Message();
                        message.what = UPDATE_TEXT;
                        // 将 Message 对象发送出去
                        handler.sendMessage(message); 
                    }
                }).start();
                break;
            default:
                break;
        }
    }
}
```

 

### 解析异步消息处理机制

Android 中的异步消息处理主要由 4 个部分组成：Message、Handler、MessageQueue 和 Looper。

> 1. Message
>
> Message 是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不用线程之间交换数据。上面使用过的 Message 的 what 字段，除此之外还可以使用 arg1 和 arg2 字段来携带一些整型数据，使用 obj 字段携带一个 Object 对象。
>
> 2. Handler
>
> 主要用于发送和处理消息的。发送消息一般使用 Handler 的 sendMessage() 方法，而经过一系列地辗转处理之后，最终会传递到 Handler 的 handlerMessage() 方法中。
>
> 3. MessageQueue 
>
> MessageQueue 是消息队列的意思，它主要用于存放所有通过 Handler 发送的消息，这部分消息会一直存在于消息队列中等待被处理。每个线程中只会有一个 MessageQueue 对象。
>
> 4. Looper
>
> Looper 是每个线程中的 MessageQueue 的管家，调用 Looper 的 loop() 方法后就会进入到一个无限循环当中，然后每当发现 MessageQueue 中存在一条消息，就会将它取出，并传递到 Handler 的 handleMessage() 方法中。每个线程也只会有一个 Looper 对象。



Message 能够从子线程进入到主线程，从不能更新 UI，整个异步消息处理的核心思想也就是如此。

### 使用 AsyncTask

为了更加方便我们在子线程中对 UI 进行操作，Android 还提供了另外一些好用的工具，AsyncTask 就是其中之一。借助 AsyncTask，即使对异步消息处理机制完全不了解，也是可以十分简单地从子线程切换到主线程。

首先看一下 AsyncTask 的基本用法，由于 AsyncTask 是一个抽象类，所以如果我们想使用它，就必须要创建一个子类去继承它。在继承时我们可以为 AsyncTask 类指定三个泛型参数，这三个参数的用途如下。

> 1. Params
>
> 在执行 AsyncTask 时需要传入的参数，可用于在后台任务中使用。
>
> 2. Progress
>
> 后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位。
>
> 3. Result
>
> 当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛用型作为返回值类型。因此一个最简单的自定义 AsyncTask 就可以写成如下方式：
>
> 



```java
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {    ... ...}
```

 这里我们把 AsyncTask 的第一个泛型参数指定为 Void，表示在执行 AsyncTask 的时候不需要传入参数给后台任务。第二个泛型参数指定为 Integer，表示使用整型数据来作为进度显示单位。第三个泛型参数指定为 BOOlean，则表示使用布尔型数据来反馈执行结果。

目前 AsyncTask 中的几个方法才能完成对任务的定制。经常需要重写的写法有以下四个。

> 1. onPreExecute()
>
> 这个方法会在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框等。
>
> 2. doInBackground(Params...)
>
> 这个方法中所有代码都会在子线程中运行，我们应该在这里处理所有的耗时任务。任务一旦完成就可以通过 return 语句来将任务的执行结果返回，如果 AsyncTask 的第三个泛型参数指定的是 Void，就可以不返回执行任务的结果。注意，在这个方法中是不可以进行 UI 操作的，如果需要更新 UI元素，比如说反馈当前任务的执行进度，可以调用 publishProgress(Progress...) 方法来完成。
>
> 3. onProgrssUpdate(Progress...)
>
> 在后台任务中调用了 publishProgress(Progress...) 方法后，这个方法就会很快被调用，方法中携带的参数就是在后台任务中传递过来的。在这个方法中可以对 UI进行操作，利用参数值就可以对界面元素进行相应的更新。
>
> 4. onPostExecute(Result)
>
> 当后台任务执行完毕并通过 return 语句进行返回时，这个方法就很快会被调用。返回的数据会作为参数传递到此方法中，可以利用返回数据来进行一些 UI 操作，比如说提醒任务执行的结果，以及关闭掉进度对话框等。



```java
public class DownloadTask extends AsyncTask<Void, Integer, Boolean> {

    @Override
    protected void onPreExecute(){
        progressDialog.show(); // 显示进度对话框
    }
    @Override
    protected Boolean doInBackground(Void... params) {
        try {
            while(true) {
                int downloadPercent = doDownload(); // 这是一个虚构的方法
                publishProgress(downloadPercent);
                if (downloadPercent >= 100) {
                    break;
                }
            }
        } catch (Exception e){
            return false
        }
        return true;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        progressDialog.setMessage("Downloaded" + values[0] + "%");
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        progressDialog.dismiss(); // 关闭进度对话框
        // 在这里提示下载结果
        ...
        
    }
}

```





## 服务的基本用法

### 启动和停止

新建一个 ServiceTest 项目，然后在这个项目中新增一个名为	MyService 的类，并让它继承自 Service，完成后的代码如下所示：

```java
public class MyService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("MyService", "onCreate executed");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d("MyService" , "onStartCommand executed");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d("MyService", "onDestroy executed");
    }
}
```

在其中加入几条日志打印信息，用于观察服务启动状态。

接下来需要修改 activity_main.xml 文件，添加启动和停止按钮

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/start_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start Service"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/stop_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stop Service"
        android:textAllCaps="false" />

</LinearLayout>
```

最后我们在 MainActivity.java 中添加按钮的逻辑

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private Button startService;
    private Button stopService;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        startService = (Button)findViewById(R.id.start_service);
        stopService = (Button)findViewById(R.id.stop_service);
        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.start_service:
                Intent startIntent = new Intent(this, MyService.class);
                startService(startIntent); // 启动服务
                break;
            case R.id.stop_service:
                Intent stopIntent = new Intent(this, MyService.class);
                stopService(stopIntent); // 停止服务
                break;
            default:
                break;
        }
    }
}
```

运行程序，先点击启动按钮，可以看到打印出了日志 

```log
/com.example.servicetest D/MyService: onCreate executed
/com.example.servicetest D/MyService: onStartCommand executed
```

并且可以在服务列表里看到，然后点击停止按钮，可以看到打印出日志

```log
/com.example.servicetest D/MyService: onDestroy executed
```

服务就已经停止了



### 活动和服务进行通信

虽然服务实在活动里启动的，但是在启动了服务之后，活动与服务就基本没什么关系了。当活动里调用了 startService() 方法来启动 MyService 这个服务，然后 MyService 的 onCreate() 和 onStartCommand() 方法就会得到执行。之后服务会一直处于运行状态，但是具体运行的是什么逻辑，活动就控制不了了。但是 Service 这个类提供一个 onBinder() 方法，可以通过创建一个专门的 Binder 对象来实现服务和活动的交互。

例：希望在 MyService 里提供一个下载功能，然后在活动中可以决定何时开始下载，以及随时查看下载进度。实现这个功能的思路是创建一个专门的 Binder 对象来对下载功能进行管理，修改 MyService 中的代码

```java
public class MyService extends Service {
    
    private DownloadBinder mBinder = new DownloadBinder();
    
    class DownloadBinder extends Binder {
        public void startDownload() {
            Log.d("MyService", "startDownload executed");
        }
        public int getProgress() {
            Log.d("MyService", "getProgress executed");
            return 0;
        }
        
    }
    
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("MyService", "onCreate executed");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d("MyService" , "onStartCommand executed");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d("MyService", "onDestroy executed");
    }
}

```

 接下来修改布局文件，添加按钮

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

	... ...
    <Button
        android:id="@+id/bind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Bind Service"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/unbind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Unbind Service"
        android:textAllCaps="false" />
</LinearLayout>
```

然后修改 MainActivity.java 

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private Button startService;
    private Button stopService;

    private MyService.DownloadBinder downloadBinder;
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder) service;
            downloadBinder.startDownload();
            downloadBinder.getProgress();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        startService = (Button)findViewById(R.id.start_service);
        stopService = (Button)findViewById(R.id.stop_service);
        Button bindService = (Button) findViewById(R.id.bind_service);
        Button unbindService = (Button) findViewById(R.id.unbind_service);

        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
        bindService.setOnClickListener(this);
        unbindService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.start_service:
                Intent startIntent = new Intent(this, MyService.class);
                startService(startIntent); // 启动服务
                break;
            case R.id.stop_service:
                Intent stopIntent = new Intent(this, MyService.class);
                stopService(stopIntent); // 停止服务
                break;
            case R.id.bind_service:
                Intent bindIntent = new Intent(this, MyService.class);
                // 绑定服务
                bindService(bindIntent, connection, BIND_AUTO_CREATE);
                break;
            case R.id.unbind_service:
                // 解绑服务
                unbindService(connection);
                break;
            default:
                break;
        }
    }
}
```

运行程序，点击按钮，可以观察到可以在活动里成功调用服务里提供的方法了。



## 服务的生命周期

与活动和碎片相似，服务也有自己的生命周期，前面使用到的 onCreate()，onStartCommand()， onBind() 和 onDestroy() 等方法都是在服务的什么周期内可能回调的方法。

一旦在项目的任何位置调用了 Context 的 StartService() 方法，相应的服务就会启动起来，并回调 onStartCommand() 方法。如果这个服务之前还没有创建过，onCreate() 方法会先于 onStartCommand() 方法执行。服务启动了之后会一直保持运行状态，直到 stopService() 或 stopSelf() 方法被调用。注意，虽然每调用一次 startService() 方法，onStartCommand() 就会执行一次，但实际上每个服务都只会存在一个实例。所有无论调用了多少次 startService() 方法，只需要调用一次 stopService() 或 stopSelf() 方法，服务就会停止下来了。

另外，还可以调用 Context 的 bindService() 来获取一个服务的持久连接，这时就会回调服务中的 onBind() 方法。类似地，如果这个服务之前还没有创建过，onCreate() 方法会先于 onBind() 方法执行。之后，调用方法可以获取到 onBind() 方法里返回的 IBinder 对象的实例，这样就能自由地和服务进行通信了。只要调用方和服务之间的连接没有断开，服务就会一直保持运行状态。

当调用了 startService() 方法后，又去调用 stopService() 方法，这时服务中的 onDestroy() 方法就会执行，表示服务已经销毁了。类似地，当调用了 bindService() 方法后，又去调用 unbindService() 方法，onDestroy() 方法也会执行，这两种情况都很好理解。但是需要注意，完全有可能一个服务既调用了 startService() 方法，又调用了 bindService() 方法的。根据 Android 系统的机制，一个服务只要被启动或者被绑定了之后，就会一直处于运行状态，必须要让以上两种条件同时不满足，服务才能被销毁。所以，这种情况下要同时调用 stopService() 和 unbindService() 方法，onDestroy() 方法才会执行。

这就是服务的整个生命周期。



## 服务的更多技巧

服务几乎都是在后台运行的，一直以来他都是默默地做着辛苦的工作。但是服务的系统优先级还是比较低的，当系统出现内存不足的情况时有可能会回收掉正在后台运行的服务。如果想保持服务一直运行，不会因为系统内存不足的原因导致被回收，就可以考虑使用前台服务。

前台服务和普通服务最大的区别就在于，他一直有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果。当然有时候你也可能不仅仅是为了放置服务被回收掉才使用前台服务的，有些项目由于特殊的需求会要求必须使用前台服务，比如说某些天气软件，服务在后台更新天气数据的同时，还会在系统状态栏一直显示当前的天气信息。

下面在原有项目的基础上将 MyService 修改为前台服务，MyService.java

```java
public class MyService extends Service {

	... ... ... 
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("MyService", "onCreate executed");
        Intent intent = new Intent(this, MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
        Notification notification = new NotificationCompat.Builder(this)
                .setContentTitle("This is content text")
                .setContentText("This is content text")
                .setWhen(System.currentTimeMillis())
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentIntent(pi)
                .build();
        startForeground(1, notification);
    }
	... ... ...
   
}

```

运行程序就实现了与天气软件相类似的前台服务了。

### 使用 IntentService

服务中的代码都是默认运行在主线程当中的，如果直接服务里去处理一些耗时的逻辑，就很容易出现 ANR （Application Not Responding） 的情况。所以在这个时候就系要用到 Android 的多线程编程的技术了，我们应该在服务的方法里开启一个子线程，然后在这里去处理那些耗时的逻辑。因此一个比较标准的服务就可以写成下面这种形式：

```java
public class MyService extends Service {
	... ...
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 处理具体的逻辑
            }
        }).start();
        return super.onStartCommand(intent, flags, startId);
    }
}
```

但是这种服务一点启动之后，就会一直处于运行状态，必须调用 stopService() 或者 stopSelf() 方法才能让服务停止下来。所以如果想要实现一个服务在执行完毕后自动停止的功能，就可以这样写：

```java
public class MyService extends Service {
	... ...
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 处理具体的逻辑
                stopSelf();
            }
        }).start();
        return super.onStartCommand(intent, flags, startId);
    }
}
```

虽然这种写法并不复杂，但是总会有一些程序员忘记开启线程，或者忘记调用 stopSelf() 方法。为了可以简单地创建一个异步的、会自动停止的服务，Android 专门提供了一个 IntentService 类，这个类就很好地解决恶前面所提到的两种尴尬。

新建一个 MyIntentService 类继承自 IntentService，代码如下

```java
public class MyIntentService extends IntentService {

    public MyIntentService() {
        super("MyIntentService"); // 调用父类的有参构造函数
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        Log.d("MyIntentService", "Thread id is"
                + Thread.currentThread().getId());
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d("MyIntentService", "onDestroy executed");
    }
}

```

接下来修改 activity_main.xml 中的代码，加入一个用于启动 MyIntentService 这个服务的按钮，如下：

```xml
    <Button
        android:id="@+id/start_intent_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start IntentService"
        android:textAllCaps="false" />
```

然后修改 MainActivity.java 中的代码

```java
Button startIntentService = 
    	(Button) findViewById(R.id.start_intent_service);
startIntentService.setOnClickListener(this);

... ...
    
case R.id.start_intent_service:
// 打印主进程 id
	Log.d("MainActivity", "Thread id is " + 
          	Thread.currentThread().getId());
	Intent intentService = 
        new Intent(this, MyIntentService.class);
	startService(intentService);
	break;
```

可以看到，我们在 Start IntentService 按钮的点击事件里面去启动 MyIntentService 这个服务，并在这里打印了一下主线程的 id，稍后用于和普通的服务没什么两样。最后不要忘记，服务都是需要在 AndroidManifest.xml 里注册，如下：

```xml
<service android:name=".MyIntentService"></service>
```

运行程序点击对应按钮就能看到 Logcat 中看到打印信息。

```
/com.example.servicetest D/MainActivity: Thread id is 1
/com.example.servicetest D/MyIntentService: Thread id is 146
/com.example.servicetest D/MyIntentService: onDestroy executed
```



## 服务的最佳实践——完整版的下载示例

这里将综合运用一下，尝试实现一个服务中经常会使用到的功能——下载。

首先需要将项目中添加依赖库，编辑 app/build.gradle 文件，在 dependencies 闭包中添加如下内容

```groovy
dependencies {

    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'com.google.android.material:material:1.2.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.1'
    implementation ("com.squareup.okhttp3:okhttp:4.9.0") // 新添加
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
}
```

接下来需要定义一个回调接口，用于对下载过程中的各种状态进行监听和回调。新建一个DownloadListenter 接口，代码如下：

```java
public interface DownloadListener {
    // 通知当前的下载进度 
    void onProgress(int progress);
    // 通知下载成功事件
    void onSuccess();
    // 通知下载失败事件
    void onFailed();
    // 通知下载暂停事件
    void onPaused();
    // 通知下载取消事件
    void onCanceled();
}
```

这里一共定义了5个回调方法。回调接口定义好了之后，下面我们就可以开始边磁轭下载功能了。这里准备用 AsyncTask 来进行处理实现，新建一个 DownloadTask 继承自 AsyncTask，代码如下：

```java
// String  表示执行 AsyncTask 时需要传入一个字符串参数给后台任务
// Integer 表示使用整型数据来作为进度显示单位
// Integer 表示用整型数据来反馈执行结果
public class DownloadTask extends AsyncTask<String, Integer, Integer> {

    public static final int TYPE_SUCCESS = 0; // 表下载成功
    public static final int TYPE_FAILED = 1;  // 表下载失败
    public static final int TYPE_PAUSED = 2;  // 表下载暂停
    public static final int TYPE_CANCELED = 3;// 表取消下载

    private DownloadListener listener;

    private boolean isCanceled = false;
    private boolean isPaused = false;

    private int lastProgress;
	// 构造函数中要求传入一个刚刚定义的 DownloadListener 参数
    // 待会就会将下载状态通过这个参数进行回调
    public DownloadTask(DownloadListener listener) {
        this.listener = listener;
    }

    @Override // 这个线程会自动开启下载任务
    protected Integer doInBackground(String... params) {
        InputStream is = null;
        RandomAccessFile saveFile = null;
        File file = null;
        try {
            // 获取 url 地址
            long downloadedLength = 0;
            String downloadUrl = params[0];
            // 解析出下载文件名
            String fileName = downloadUrl.substring(downloadUrl.lastIndexOf("/"));
            // 本地保存下载的目录
            String directory = Environment.getExternalStoragePublicDirectory
                    (Environment.DIRECTORY_DOWNLOADS).getPath();
            file = new File(directory + fileName);
            // 判断文件是否存在
            if(file.exists()) {
                downloadedLength = file.length();
            }
            // 下载
            long contentLength = getContentLength(downloadUrl);
            if(contentLength == 0)
                return TYPE_FAILED;
            if(contentLength == downloadedLength)
                return TYPE_SUCCESS;
            
            // 断点续传下载请求
            OkHttpClient client = new OkHttpClient();
            // 添加头告知服务器想从那个字节开始下载
            Request request = new Request.Builder()
                    .url(downloadUrl)
                    .addHeader("RANGE", "bytes=" + downloadedLength + "-")
                    .build();
            Response response = client.newCall(request).execute();
            if(response != null) {
                is = response.body().byteStream();
                saveFile = new RandomAccessFile(file, "rw");
                byte[] b = new byte[512];
                int total = 0;
                int len;
                
                // 判断是否有暂停和取消取消
                while ((len = is.read(b)) != -1) {
                    if(isCanceled) {
                        return  TYPE_CANCELED;
                    } else if (isPaused) {
                        Log.d("Download", "下载暂停 " + 
                              Thread.currentThread().getId());
                        return TYPE_PAUSED;
                    } else {
                        total += len;
                        saveFile.write(b, 0, len);
                        // 通知进度条更新
                        int progress = (int) ((total + downloadedLength) * 
                                              100 / contentLength);
                        // 通知下载进度
                        publishProgress(progress);
                    }
                }
            }
            response.body().close();
            return TYPE_SUCCESS;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if(is != null) {
                    is.close();
                }
                if(saveFile != null){
                    saveFile.close();
                }
                if(isCanceled && file != null){
                    file.delete();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return TYPE_FAILED;
    }

    @Override // 根据参数传入的下载状态进行回调
    protected void onPostExecute(Integer status){
        switch (status) {
            case TYPE_SUCCESS:
                listener.onSuccess();
                break;
            case TYPE_FAILED:
                listener.onFailed();
                break;
            case TYPE_CANCELED:
                listener.onCanceled();
                break;
            case TYPE_PAUSED:
                listener.onPaused();
                break;
            default:
                break;
        }
    }

    @Override // 获取下载进度与上一次进行比对 通知更新进度
    public void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
        int progress = values[0];
        if(progress >= lastProgress) {
            listener.onProgress(progress);
            lastProgress = progress;
        }
    }


    public void cancelDownload(){
        isCanceled = true;
    }


    public void pauseDownload(){
        isPaused = true;
    }
	
    // 下载的具体实现
    private long getContentLength(String downloadUrl) throws IOException {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(downloadUrl)
                .build();
        Response response = client.newCall(request).execute();
        if(response != null && response.isSuccessful()) {
            long contentLength = response.body().contentLength();
            response.close();
            return contentLength;
        }
        return 0;
    }
}
```

具体的下载功能完成了，下面为了保证 DownloadTask 可以一直在后台运行，还需要创建一个下载的服务，新建 DownloadService

```java
public class DownloadService extends Service {

    private DownloadTask downloadTask;
    private String downloadUrl;
    private DownloadListener listener = new DownloadListener() {
        @Override
        public void onProgress(int progress) {
            getNotificationManager().notify(1, getNotification("Downloading...", progress));
        }

        @Override
        public void onSuccess() {
            downloadTask = null;
            // 下载成功时将前台服务通知关闭，并创建一个下载成功通知
            stopForeground(true);
            getNotificationManager().notify(1, getNotification("Download Success", -1));
            Toast.makeText(DownloadService.this, "Download Success",
                    Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onFailed() {
            downloadTask = null;
            // 下载失败时将前台服务通知关闭，并创建一个下载失败通知
            stopForeground(true);
            getNotificationManager().notify(1, getNotification("Download Failed", -1));
            Toast.makeText(DownloadService.this, "Download Failed",
                    Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onPaused() {
            downloadTask = null;
            Toast.makeText(DownloadService.this, "Download Paused",
                    Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onCanceled() {
            downloadTask = null;
            stopForeground(true);
            Toast.makeText(DownloadService.this, "Download Canceled",
                    Toast.LENGTH_SHORT).show();
        }
    };

    private DownloadBinder mBinder = new DownloadBinder();

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    class DownloadBinder extends Binder {
        public void startDownload(String url) {
            if(downloadTask == null){
                downloadUrl = url;
                downloadTask = new DownloadTask(listener);
                downloadTask.execute(downloadUrl);
                startForeground(1, getNotification("Downloading...", 0));
                Toast.makeText(DownloadService.this, "Downloading...",
                        Toast.LENGTH_SHORT).show();
            }
        }
        public void pauseDownload() {
            if(downloadTask != null) {
                downloadTask.pauseDownload();
                Toast.makeText(DownloadService.this, "Download Paused",
                        Toast.LENGTH_SHORT).show();
            }
        }
        public void cancelDownload() {
            if(downloadTask != null) {
                downloadTask.cancelDownload();
            } else {
                if(downloadUrl != null) {
                    String fileName = downloadUrl.substring(downloadUrl.lastIndexOf("/"));
                    String directory = Environment.getExternalStoragePublicDirectory(
                            Environment.DIRECTORY_DOWNLOADS).getPath();
                    File file = new File(directory + fileName);
                    if(file.exists()) {
                        file.delete();
                    }
                    getNotificationManager().cancel(1);
                    stopForeground(true);
                    Toast.makeText(DownloadService.this, "Download Canceled",
                            Toast.LENGTH_SHORT).show();

                }
            }
        }
    }

    public NotificationManager getNotificationManager() {
        return (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
    }

    public Notification getNotification(String title, int progress) {
        Intent intent = new Intent(this, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentTitle(title);
        builder.setContentIntent(pendingIntent);
        if (progress > 0) {
            builder.setContentText(progress + "%");
            builder.setProgress(100, progress, false);
        }
        return builder.build();
    }

}
```



下载服务也已经成功实现，后端的工作基本完成，现在来编写前端部分，修改主界面布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/start_download"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="START" />
    <Button
        android:id="@+id/pause_download"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="PAUSE" />
    <Button
        android:id="@+id/cancel_download"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="CANCEL" />

</LinearLayout>
```

然后修改 MainActivity.java 文件

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    private DownloadService.DownloadBinder downloadBinder;
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (DownloadService.DownloadBinder) service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button start = (Button) findViewById(R.id.start_download);
        Button pause = (Button) findViewById(R.id.pause_download);
        Button cancel = (Button) findViewById(R.id.cancel_download);

        start.setOnClickListener(this);
        pause.setOnClickListener(this);
        cancel.setOnClickListener(this);
        Intent intent = new Intent(this, DownloadService.class);
        startService(intent);
        bindService(intent, connection, BIND_AUTO_CREATE);
        if(ContextCompat.checkSelfPermission(MainActivity.this,
                Manifest.permission.WRITE_EXTERNAL_STORAGE) !=
                PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(MainActivity.this,
                    new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
        }
    }

    @Override
    public void onClick(View v) {
        if(downloadBinder == null)
            return;
        switch (v.getId()) {
            case R.id.start_download:
                String url = "https://download.test.com/download.exe";
                downloadBinder.startDownload(url);
                break;
            case R.id.pause_download:
                downloadBinder.pauseDownload();
                break;
            case R.id.cancel_download:
                downloadBinder.cancelDownload();
                break;
            default:
                break;
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions,
                                           int[] grantResults) {
        switch (requestCode) {
            case 1:
                if(grantResults.length > 0 && grantResults[0] !=
                        PackageManager.PERMISSION_GRANTED) {
                    Toast.makeText(this, "denied permission",
                            Toast.LENGTH_SHORT).show();
                    finish();
                }
                break;
            default:
                break;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(connection);
    }
}
```

最后添加权限

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicebestpractice">
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.ServiceBestPractice">
        <service
            android:name=".DownloadService"
            android:enabled="true"
            android:exported="true"></service>

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>

</manifest>
```

运行程序，即可测试。





## END

