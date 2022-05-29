# Android 学习 04

【主要内容】简单和常用的控件简介。

[TOC]



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

































