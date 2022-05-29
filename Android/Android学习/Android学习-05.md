# Android 学习 05

【主要内容】碎片（Fragment）

[TOC]



【碎片（Fragment）】

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

















