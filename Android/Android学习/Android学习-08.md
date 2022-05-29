# Android 学习 08

【跨程序共享数据】探究内容提供器

[TOC]



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
ContentValues values = new ContentValues();
values.put("column1", "text");
values.put("column2", 1);
getContentResolver().insert(uri, values);
```

* **改**

```java
ContentValues values = new ContentValues();
values.put("column1", "");
getContentResolver().update(uri, values, "column1 = ? and column2 = ?",
                           new String[] { "text", "1" });
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






























