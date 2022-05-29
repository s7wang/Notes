# Linphone 结构分析





## LinphoneManager



```java
public class LinphoneManager implements SensorEventListener
```

implements 声明自己使用一个或多个接口；extends 是继承某个类来创建一个类的子类。extends  一般是先继承父类的方法，也可以重写父类的方法；implements 是实现多接口，接口的方法一般为空的，必须重写才能使用。extends 是继承父类，只要那个类不是声明为 final 或者那个类定义为 abstract 的就能继承，Java 不支持多重继承，但是可以用接口来实现这样就能实现多个接口了。

SensorEventListener.java 传感器事件监听器接口

```java
public interface SensorEventListener {
    void onSensorChanged(SensorEvent var1);

    void onAccuracyChanged(Sensor var1, int var2);
}
```



* 第一组变量

```java
private final String mBasePath;
private final String mRingSoundFile, mRingBackSoundFile;
private final String mCallLogDatabaseFile;
private final String mFriendsDatabaseFile;
private final String mUserCertsPath;
```




* 第二组变量

```java
private final Context mContext;
private AndroidAudioManager mAudioManager;
private CallManager mCallManager;
private ContactManager mContactManager;
private GroupManager mGroupManager;
private ChatManager mChatManager;
private LoginManager mLoginManager;
private List<LinphoneReadyListener> mReadyListeners = new ArrayList<LinphoneReadyListener>();
```

各种管理器，负责管理各种基础的事务



* 第三组变量

```java
private final PowerManager mPowerManager;
private final ConnectivityManager mConnectivityManager;
private TelephonyManager mTelephonyManager;
private PhoneStateListener mPhoneStateListener;
private WakeLock mProximityWakelock;
private final SensorManager mSensorManager;
private final Sensor mProximity;
// private final MediaScanner mMediaScanner;
private Timer mTimer, mAutoAnswerTimer;
```

其他管理器。



* 第四组变量

```java
    private final LinphonePreferences mPrefs;
    private Core mCore = null;
    private CoreListenerStub mCoreListener;
    private AccountCreator mAccountCreator;
    private AccountCreatorListenerStub mAccountCreatorListener;

    private boolean mExited;
    private boolean mCallGsmON;
    private boolean mProximitySensingEnabled;
    private boolean mHasLastCallSasBeenRejected;
    private Runnable mIterateRunnable;
```





* 第五组

单例模式的实例，静态全局变量，用于保证这个类仅有一个实例并提供全局访问站点。

```java
private static LinphoneManager manager = null;
```





* 第六组

私有化的构造函数，初始化各种储存文件和各种传感器，其中的核心为 mCoreListener 为核心监听器

```java
private LinphoneManager(Context c) {
    mExited = false;
    mContext = c;
    mBasePath = c.getFilesDir().getAbsolutePath();
    mCallLogDatabaseFile = mBasePath + "/linphone-log-history.db";
    mFriendsDatabaseFile = mBasePath + "/linphone-friends.db";
    mRingSoundFile = mBasePath + 
        "/share/sounds/linphone/rings/notes_of_the_optimistic.mkv";
    mRingBackSoundFile = mBasePath + "/share/sounds/linphone/ringback.wav";
    mUserCertsPath = mBasePath + "/user-certs";

    String basePath = c.getFilesDir().getAbsolutePath();
    Factory.instance().setLogCollectionPath(basePath);

    mPrefs = LinphonePreferences.instance();
    mPrefs.setContext(mContext);
	// 电源管理器
    mPowerManager = 
        (PowerManager) c.getSystemService(Context.POWER_SERVICE);
    // 网络连接管理器
    mConnectivityManager = 
        (ConnectivityManager) c.getSystemService(Context.CONNECTIVITY_SERVICE);
    //  传感器？
    mSensorManager = 
        (SensorManager) c.getSystemService(Context.SENSOR_SERVICE);
    // 代理管理器
    mProximity = 
        mSensorManager.getDefaultSensor(Sensor.TYPE_PROXIMITY);
    
    mHasLastCallSasBeenRejected = false;
	// 各种管理器
    mCallManager = new CallManager(c);
    mChatManager = new ChatManager(c);
    mContactManager = new ContactManager(c);
    mGroupManager = new GroupManager(c);
    mLoginManager = new LoginManager(c);
    
    File f = new File(mUserCertsPath);
    if (!f.exists()) {
        if (!f.mkdir()) {
            Log.e("[Manager] " + mUserCertsPath + " can't be created.");
        }
    }
	// 证书？那些乱七八糟的
    try {
        // Let's copy some RAW resources to the device
        // The default config file must only be installed 
        // once (the first time)
        copyIfNotExist(org.linphone.sample.R.raw.linphonerc_default, 
                       basePath + "/.linphonerc");
        // The factory config is used to override any other setting, 
        // let's copy it each time
        copyFromPackage(org.linphone.sample.R.raw.linphonerc_factory, 
                        "linphonerc");
    } catch (IOException ioe) {
        Log.e(ioe);
    }

    // mMediaScanner = new MediaScanner(c);

    mCoreListener =
        new CoreListenerStub() {};
}
```





* 第七组

```java

mCoreListener =
    new CoreListenerStub() {
    @SuppressLint("Wakelock")
    @Override
    public void onCallStateChanged(
        final Core core,
        final Call call,
        final State state,
        final String message) {
        Log.i("[Manager] Call state is [", state, "]");
        // 来电 状态
        if (state == State.IncomingReceived
            && !call.equals(core.getCurrentCall())) {
            if (call.getReplacedCall() != null) {
                // attended transfer will be accepted automatically.
                return;
            }
        }
		// 占线
        if ((state == State.IncomingReceived || state == State.IncomingEarlyMedia)
            && getCallGsmON()) {
            if (mCore != null) {
                call.decline(Reason.Busy);
            }
        } else if (state == State.IncomingReceived
                   && (LinphonePreferences.instance().isAutoAnswerEnabled())
                   && !getCallGsmON()) {
            TimerTask lTask =
                new TimerTask() {
                @Override
                public void run() { // 接电话
                    if (mCore != null) {
                        if (mCore.getCallsNb() > 0) {
                            mCallManager.acceptCall(call);
                            mAudioManager.routeAudioToEarPiece();
                        }
                    }
                }
            };
            
            mAutoAnswerTimer = new Timer("Auto answer");
            mAutoAnswerTimer.schedule(lTask, mPrefs.getAutoAnswerTime());
        // 结束或异常
        } else if (state == State.End || state == State.Error) {
            if (mCore.getCallsNb() == 0) {
                // Disabling proximity sensor
                enableProximitySensing(false);
            }
        } else if (state == State.UpdatedByRemote) {
            // If the correspondent proposes video while audio call
            boolean remoteVideo = call.getRemoteParams().videoEnabled();
            boolean localVideo = call.getCurrentParams().videoEnabled();
            boolean autoAcceptCameraPolicy =
                LinphonePreferences.instance()
                .shouldAutomaticallyAcceptVideoRequests();
            if (remoteVideo
                && !localVideo
                && !autoAcceptCameraPolicy
                && mCore.getConference() == null) {
                call.deferUpdate();
            }
        }
    }

    // 全局状态监听
    @Override
    public void onGlobalStateChanged(Core core, GlobalState state, String message) {
        Log.i("[Context] Global state is [", state, "]");

        if (state == GlobalState.On) {
            for (LinphoneReadyListener listener : mReadyListeners) {
                listener.onLinphoneReady();
            }
        }
    }
	
    // 版本更新？
    @Override
    public void onVersionUpdateCheckResultReceived(
        Core core,
        VersionUpdateCheckResult result,
        String version,
        String url) {
        if (result == VersionUpdateCheckResult.NewVersionAvailable) {

        }
    }
	
    // 打开通知接收
    @Override
    public void onNotifyReceived(Core lc, Event evt, String var3, Content content) {
        Log.i("yhw:onNotifyReceived content =" + content.getStringBuffer());
        Log.i("yhw:onNotifyReceived event =" + evt.getName() + " var3 = " + var3);
        if (var3 != null && var3.equalsIgnoreCase("presence")) {
            String contentString = content.getStringBuffer();
            StatusSubscribe status = 
                XmlUtils.parseStatusSubscribeXml(contentString);
            List<OnlineStatus> onlineStatuses = status.getList();
            getContactManager().onPresenceReceived(onlineStatuses);
        } else {

        }
    }
	// 注册状态改变
    @Override
    public void onRegistrationStateChanged(
        Core core, ProxyConfig cfg, 
        RegistrationState state, 
        String message) {
        getLoginManager().onRegisterState(cfg, state, message);
    }
	
    // 消息接收
    @Override
    public void onMessageReceived(Core lc, ChatRoom room, ChatMessage message) { ... }
};

```



消息接收

```java
@Override
public void onMessageReceived(Core lc, ChatRoom room, ChatMessage message) {
    room.markAsRead();
    Log.i("yhw:onMessageReceived");
    Address address = room.getPeerAddress();
    Log.i("yhw:room  address  = " + address.asStringUriOnly());
    address = message.getFromAddress();
    String contentType = message.getContentType();
    String content = message.getTextContent();
    String servInfo = message.getCustomHeader("ServInfo");
    if (servInfo.equalsIgnoreCase("resource-lists")) {
        String type = XmlUtils.getXmlType(content);
        if (type != null) {
            if (type.equalsIgnoreCase(IXmlCommon.E_RESPONSE)) {
                BaseResponse response = 
                    XmlUtils.parseResponseXml(content);

                if (response != null) {
                    Log.i("yhw:response = " + response.toString());
                    String cmdType = response.getCmdType();
                    if (cmdType != null) {
                        if (cmdType.equalsIgnoreCase("Global")) {
                            int action = response.getAction();
                            QueryContactResponse response1 = 
                                XmlUtils.parseQueryContactResponse(content);
                            Log.i("yhw:response1 = " + response1.toString());
                            getContactManager().onQueryContactResponse(response1);
                        } else if (cmdType.equalsIgnoreCase("Users")) {
                            int action = response.getAction();
                        } else if (cmdType.equalsIgnoreCase("Groups")) {
                            int action = response.getAction();
                            int sn = response.getSn();
                            if (sn == SnUtils.queryGroupSN) {
                                getGroupManager().onQueryGroupResponse(
                                    	(QueryGroupResponse) response);
                            } else {
                                getGroupManager().onCreateGroupResponse(
                                    	(CreateGroupResponse) response);
                            }
                        } else if (cmdType.equalsIgnoreCase("Wireless")) {
                            //int action = response.getAction();
                            int sn = response.getSn();
                            if (sn == SnUtils.queryWirelessSN) {
                                QueryWirelessResponse response1 = 
                                    XmlUtils.parseQueryWirelessResponse(content);
                                getContactManager().onQueryWirelessResponse(response1);
                            }
                        }
                    } else {
                        Log.e("yhw: cmdType is null!");
                    }
                } else {
                    Log.e("yhw: parse response error! response is null!");
                }
            } else if (type.equalsIgnoreCase("Control")) {

                int controlType = XmlUtils.getControlType(content);
                if (controlType == 0) {
                    CreateGroupControl control = 
                        XmlUtils.parseCreateGroupControl(content);
                    if (control != null) {
                        getGroupManager().onCreateGroupRequest(control);
                    }

                } else if (controlType == 1) {
                    DeleteGroupControl control = 
                        XmlUtils.parseDeleteGroupControl(content);
                    if (control != null) {
                        getGroupManager().onDeleteGroupRequest(control);
                    }

                } else if (controlType == 2) {
                    ModifyGroupControl control = 
                        XmlUtils.parseModifyGroupControl(content);
                    if (control != null) {
                        getGroupManager().onModifyGroupRequest(control);
                    }

                } else if (controlType == 3) {
                    MemberOperateControl control = 
                        XmlUtils.parseMemberOperateControl(content);
                    if (control != null) {
                        getGroupManager().onAddGroupMemberRequest(control);
                    }

                } else if (controlType == 4) {
                    MemberOperateControl control = 
                        XmlUtils.parseMemberOperateControl(content);
                    if (control != null) {
                        getGroupManager().onRemoveGroupMemberRequest(control);
                    }

                } else if (controlType == 5) {
                    MemberOperateControl control = 
                        XmlUtils.parseMemberOperateControl(content);
                    if (control != null) {
                        getGroupManager().onModifyGroupMemberRequest(control);
                    }

                } else {
                    Log.e("yhw: unsupport control type!");
                }

            } else {
                Log.e("yhw: xml is unsupport type!");
            }
        } else {
            Log.e("yhw: xml type is null!");
        }
    } else if (servInfo.equalsIgnoreCase("text")) {
        //Single chat message
        List<MessageListener> listeners = getChatManager().getListeners();
        String msgString = message.getTextContent();
        if (msgString.startsWith("text:")) {
            msgString = msgString.substring(5);
            for (MessageListener listener : listeners) {
                listener.onMessageReceived(
                    room.getPeerAddress().asStringUriOnly(), 
                    message.getMessageId(), 
                    msgString);
            }

        } else if (msgString.startsWith("file:")) {
            msgString = msgString.substring(5);
            LinphoneManager.getInstance(null)
                .getChatManager()
                .onFileMessageReceived(message.getMessageId(), msgString);
        } else {
            for (MessageListener listener : listeners) {
                listener.onMessageReceived(
                    room.getPeerAddress().asStringUriOnly(), 
                    message.getMessageId(), 
                    msgString);
            }
        }
    } else if (servInfo.equalsIgnoreCase("group-text")) {
        //Group chat message
        String msgString = message.getTextContent();
        Address toAddress = message.getToAddress();
        if (msgString.startsWith("text:")) {
            msgString = msgString.substring(5);
            List<MessageListener> listeners = 
                getChatManager().getListeners();
            for (MessageListener listener : listeners) {
                listener.onGroupMessageReceived(
                    room.getPeerAddress().asStringUriOnly(), 
                    message.getMessageId(), 
                    msgString);
            }
        } else if (msgString.startsWith("file:")) {
            msgString = msgString.substring(5);
            LinphoneManager.getInstance(null)
                .getChatManager()
                .onGroupFileMessageReceived(message.getMessageId(), msgString);
        } else {
            List<MessageListener> listeners = getChatManager().getListeners();
            for (MessageListener listener : listeners) {
                listener.onGroupMessageReceived(
                    room.getPeerAddress().asStringUriOnly(), 
                    message.getMessageId(), 
                    msgString);
            }
        }
    } else if (contentType.equalsIgnoreCase("text/plain")) {
        List<MessageListener> listeners = getChatManager().getListeners();
        for (MessageListener listener : listeners) {
            listener.onMessageReceived(
                room.getPeerAddress().asStringUriOnly(), 
                message.getMessageId(), 
                message.getTextContent());
        }
    } else {
        Log.e("yhw: unsupport content type  " + contentType);
    }
}
```



## LinphonePreferences

功能：设置用户信息、设置端口号、显示名称、设置密码、设置代理、设置编码、设置编码速率、设置DMTF等、设置加密解密、设置是否使用 ipv6 、设置 tunnel设置相机。































