# Android 学习 01

【主要内容】Android 项目结构简介。

[TOC]

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



