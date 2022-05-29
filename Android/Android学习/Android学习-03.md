# Android 学习 03

【主要内容】活动生命周期。

[TOC]



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

































