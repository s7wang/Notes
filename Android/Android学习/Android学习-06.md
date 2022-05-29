# Android 学习 06

【主要内容】全局喇叭详解广播机制：为了方便于进行系统级别的消息通知，Android 也引入了一套类似广播消息机制。

[TOC]

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























