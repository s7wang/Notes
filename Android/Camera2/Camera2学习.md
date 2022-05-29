# Android Camera2 学习



由于最近需要写一个推流的module，需要调用摄像头显示预览画面，并把摄像头的数据推送到服务器上，所以开始学习了Android的摄像头相关使用。由于之前没有接触过Android开发，尤其是摄像头这部分，所以大部分内容都是在网上找的相关教程，和官方文档。但是大多是教程都做得比较专业，我这种没怎么学过的看起来就很乱，不知道到底要干什么每一部分都是在干什么；而官方文档 Camera2的部分有比较简略，没有例子。所以，打算写一篇简单易懂的Camera2使用流程。



## 最简单的摄像头调用

首先在不考虑多线程调用和其它程序调用摄像头的前提下，只考虑当前活动调取一个未被占用的摄像头，并把预览画面显示在屏幕上的功能。由于本人没有使用过Camera（被Google弃用了），所以我不知道原来是怎么调用的，现在我先上流程，调用摄像头需要什么，怎么调起来。

最最最开始，需要一个界面去显示预览界面，很简单的一个界面：

* fragment.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextureView
        android:id="@+id/cameraPreview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>


</RelativeLayout>
```

里面只有一个 TextureView。然后，调取摄像头需要一个CameraManager来统一管理，CameraManager有一个`openCamera`方法，负责打开指定摄像头，并调取设备状态`CameraDevice.StateCallback`，还有一个`handler`参数（暂时用不到先不管），然后调取设备状态的参数会获取摄像头设备然后创建一个捕获会话`createCaptureSession`，其中第一个参数是一个surface列表可以把需要显示的界面统统打包进去一起传进去，第二个是捕获会话的回调`CameraCaptureSession.StateCallback`，第三个还是`handler`（暂时用不到先不管）,在捕获会话的回调`CameraCaptureSession.StateCallback`中，设置捕获请求 并添加输出界面`mCaptureSession.setRepeatingRequest`。当然别忘了在TextureView的`onSurfaceTextureAvailable`中去进行以上操作。

简单的流程如下

```
(CameraManager) manager 
			└ manager.openCamera(mCameraId, mDeviceStateCallback, null);
			(CameraDevice) CameraDevice --------------┘
					|
					|
			mCameraDevice.createCaptureSession(asList(mPreviewSurface),
    				mCaptureSessionStateCallback, null);
								|
            (CameraCaptureSession) mCaptureSession 
            					|
            			mCaptureSession.setRepeatingRequest((CaptureRequest) request,
            						null, null);
```

下面附上带注释的源码，结合上面的流程下面的代码将更加清楚。Camera2Fragment.java

```java
/************************************
 * @Class: Camera2Fragment
 * @Author: wangs7
 * @Date: 2021/9/10 15:25
 * @Version: 1.0
 * @Descriptin: TODO
 ************************************/
public class Camera2Fragment extends Fragment {
    /**
     * 调用摄像头并显示预览主要分为以下几个步骤
     * 1. 实例化 TextureView 并设置监听器
     * 2. 监听器中，若控件可获取那么获取 CameraManager
     * 3. 使用 CameraManager 打开指定ID的摄像头，并传入一个设备状态的回调参数
     * 4. 在设备状态的回调中，先获取摄像头设备，然后传入设置好的显示实例，并添加会话状态回调
     * 5. 在会话状态回调中设置捕获请求并构建实例，添加输出界面。
     * */
    public static Camera2Fragment newInstance() {
        return new Camera2Fragment();
    }

    private static final String TAG = Camera2Fragment.class.getSimpleName();
    private View view;
    private TextureView mTextureView;
    private Surface mPreviewSurface;
    private static final int REQUEST_CAMERA_PERMISSION = 1;

    private String mCameraId = String.valueOf(CameraCharacteristics.LENS_FACING_FRONT); //前置摄像头ID
    private CameraDevice mCameraDevice; //摄像机硬件设备
    private CameraManager manager; //摄像头管理器 单例 全局唯一
    private CameraCaptureSession mCaptureSession; //捕获会话
    /* 5. 在会话状态回调中设置捕获请求并构建实例，添加输出界面。 */ //捕获会话的回调
    private CameraCaptureSession.StateCallback mCaptureSessionStateCallback 
        	= new CameraCaptureSession.StateCallback() {
        @Override
        public void onConfigured(@NonNull CameraCaptureSession session) {
            mCaptureSession = session;
            try {
                //请求构造器
                CaptureRequest.Builder requestBuilder = 
                    mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
                //并添加输出界面
                requestBuilder.addTarget(mPreviewSurface);
                //设置捕获请求
                mCaptureSession.setRepeatingRequest(requestBuilder.build(), null, null);

            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onConfigureFailed(@NonNull CameraCaptureSession session) {

        }
    };
	/* 4. 在设备状态的回调中，先获取摄像头设备，然后传入设置好的显示实例，并添加会话状态回调 */
    private CameraDevice.StateCallback mDeviceStateCallback 
        	= new CameraDevice.StateCallback() {
        @Override
        public void onOpened(@NonNull CameraDevice camera) {
            mCameraDevice = camera;
            try { 
                //创建捕获会话
                mCameraDevice.createCaptureSession(asList(mPreviewSurface), 
                                                   mCaptureSessionStateCallback, null);
            } catch (CameraAccessException e){
                e.printStackTrace();
            }
        }

        @Override
        public void onDisconnected(@NonNull CameraDevice camera) {

        }

        @Override
        public void onError(@NonNull CameraDevice camera, int error) {

        }
    };

    /* 2. 监听器中，若控件可获取那么获取 CameraManager */ //控件监听器
    private final TextureView.SurfaceTextureListener mSurfaceTextureListener
            = new TextureView.SurfaceTextureListener() {

        @Override //获取控件执行动作 打开摄像头
        public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
            mPreviewSurface = new Surface(surface);
            //获取管理器
            manager = (CameraManager) getActivity().getSystemService(CAMERA_SERVICE);
            //权限校验
            if (ContextCompat.checkSelfPermission(getActivity(), Manifest.permission.CAMERA)
                    != PackageManager.PERMISSION_GRANTED) {
                requestCameraPermission();
                return;
            }
            try {
                /* 3. 使用 CameraManager 打开指定ID的摄像头，并传入一个设备状态的回调参数 */ //打开摄像头
                manager.openCamera(mCameraId, mDeviceStateCallback, null);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {

        }

        @Override
        public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
            return false;
        }

        @Override
        public void onSurfaceTextureUpdated(SurfaceTexture surface) {

        }
    };

    //@Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        view = inflater.inflate(R.layout.fragment_display, container, false);
        return view;
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
		/* 1. 实例化 TextureView 并设置监听器 */ //实例化
        mTextureView = (TextureView) view.findViewById(R.id.cameraPreview);
		// 旋转 180
        mTextureView.setRotation(180.0f);


    }

    @Override
    public void onResume() {
        super.onResume();
        if (mTextureView.isAvailable()) {
            //待完成
        } else { /* 1. 实例化 TextureView 并设置监听器 */ //绑定监听器
            mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);
        }
    }

    private void requestCameraPermission() {
        if (shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
            Log.d(TAG, "Camera permission is ready.--------------------------");
        } else {
            requestPermissions(new String[]{Manifest.permission.CAMERA}, REQUEST_CAMERA_PERMISSION);
        }
    }



    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        if (requestCode == REQUEST_CAMERA_PERMISSION) {
            if (grantResults.length != 1 || grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                Log.e(TAG, "Camera permission request failed.-------------------------");
            }
        } else {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        }

    }

}
```

主活动调用 Fragment。Camera2Activity.java

```java
public class Camera2Activity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        requestCameraPermission(); //要摄像头权限
        setContentView(R.layout.activity_camera2);
        if (null == savedInstanceState) {
            // activity_camera2 布局中直接设置一个容器，启动后被Fragment替换即可。
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.container, Camera2Fragment.newInstance())
                    .commit();
        }
    }
    
    private void requestCameraPermission() {
        if (shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {

        } else {
            requestPermissions(new String[]{Manifest.permission.CAMERA}, 1);
        }
    }
}
```

至此，整个简单的Demo就能够实现利用Camera2调取摄像头，并实时显示预览画面。

### 部分问题

1. 注意 TextureView 的实例化结果校验，我就遇到过 TextureView 莫名实例化失败，提示空指针尝试调用方法的提示，后面莫名其妙的好了。。。。
2. 一定在应用启动的时候就要摄像头权限。
3. TextureView 的 surface 显示画面是倒的，可以把 TextureView 转 180 度。 





## 推流实现



