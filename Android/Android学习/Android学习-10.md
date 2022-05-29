# Android 学习 10

【网络技术】WebView、网络请求、格式解析

[TOC]



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

























