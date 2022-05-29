# Android 学习 02

【主要内容】简单的活动创建。

[TOC]

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
// 调用 setOnClickListener 为按钮注册一个监听器，点击按钮时会执行 onClick() 方法
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) { // 重写 onClick() 方法
        finish(); //销毁当前活动
    }
});
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













