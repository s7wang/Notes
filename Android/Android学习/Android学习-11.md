# Android 学习 11

【服务】

[TOC]



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
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {
    ... ...
}
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

























