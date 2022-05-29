# Android 学习 07

【数据存储全方案】详解持久化技术

[TOC]



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











