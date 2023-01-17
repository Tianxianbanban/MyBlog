## Camera2



Android 5.0 之后，相机 API 用的是 `android.hardware.camera2` 包下的内容了 。



**基本原理**

采用了**管道**的概念，将Camera Device相机设备与Android Device安卓设备连接起来，建立一个会话，所有的预览、拍照请求，都在这个会话基础上发送给Camera Device，而Camera Device则返回CameraMetadata数据给Android Device。



**调用过程以及关键类**

+ 检查相机权限（android.permission.CAMERA）

+ 创建预览类（ SurfaceView 或者 TextureView ）

+ 打开相机（ CameraManager.openCamera ）

  **CameraManager：摄像头管理器**，来打开和关闭摄像头。在Camera打开之前主要操作CameraManager，打开后主要操作CameraCaptureSession。CameraManager可以通getSystemService(Context.CAMERA_SERVICE)获取。

  关于相机设备，可以通过**CameraManager的getCameraIdList()**方法来获取到所有摄像头设备的id列表。

  CameraCharacteristics：描述了摄像头的一些属性，通过**CameraManager的getCameraCharacteristics(@NonNull String cameraId)**方法来获取。获取Camera属性信息，配置Camera输出，如FPS每秒传输帧数，大小，旋转等等。

+ 相机回调（ CemeraDevice.StateCallback ）

  在**CameraManager的openCamera(cameraId, stateCallback, backgroundHandler)方法就可以打开摄像头了**。其中CameraDevice.StateCallback callback：摄像头打开的相关回调，里面定义了打开，关闭以及出错等各种回调方法，我们可以在里面做对应的操作。 就在这传入的这个**回调当中的onOpened方法就代表摄像头已经打开了**，就能在这里获取到**CameraDevice**，并且准备开始预览或者拍照，当然这要建立在CameraDevice创建的CameraCaptureSession会话当中。

  **CameraDevice：描述系统摄像头**，用于创建CameraCaptureSession和关闭摄像头。

+ 创建 CameraCaptureSession 会话（ createCaptureSession ）

  **CameraCaptureSession：相机捕获会话**，**通过CameraCaptureSession是建立了一个和Camera硬件设备的会话通道**，就可以在这个会话的基础上向Camera发送请求，获取图像。CameraDevice可以通过createCaptureSession(Arrays.asList(mSurfaceHolder.getSurface(), mImageReader.getSurface())，CameraCaptureSession.StateCallback()，handler）方法来创建会话了，里面需要传入的是需要输出到的Surface列表，还有**会话状态的回调**，以及一个用来区别哪个callback应该被回调的handler。其中**CameraCaptureSession.StateCallback里的回调方法里onConfigured**方法回调到这里是摄像头完成配置，可以获取CameraCaptureSession，并且处理Capture请求了；相应onConfigureFailed是配置失败的情况。

+ 创建一个进行预览的请求（ CaptureRequest.Builder ）

  处理Capture请求，是调用**CameraCaptureSession的setRepeatingRequest(captureRequest, callback, childHandler)方法控制预览**；**capture(mCaptureRequest, callback, childHandler)方法则控制拍照**。**CaptureRequest代表了一次捕获请求**，是预览或拍照都需要参数，其中包含了捕获图片的参数设置，比如对焦模式、曝光模式等，就是具体设置还是对CaptureRequest.Builder对象进行操作。CaptureRequest的创建需要**CaptureRequest.Builder** ，CaptureRequest.Builder的创建需要使用到**CameraDevice的createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW)**，其中传入的是CameraDevice中设置好的常量，代表了**请求类型**，包括创建预览的请求CameraDevice.TEMPLATE_PREVIEW以及其它，比如静态图像的捕获、创建视频录制的请求等等。而关于**CaptureRequest.Builder的addTarget(mSurfaceHolder.getSurface())**方法是预览请求中设置输出的Surface。另外通过CaptureRequest.Builder可以进行对焦模式、曝光模式等等的设置。然后**CaptureRequest.Builder的build方法就可以创建CaptureRequest对象了**。

+ 在会话中发出拍照的请求（ capture ）

  调用**CameraCaptureSession的capture(captureRequest, callback, childHandler)方法则控制拍照**。

  与setRepeatingRequest方法传入相同参数，CaptureRequest对象的设置也是相同的。

+ 在会话的回调类中处理发出请求的结果（ CameraCaptureSession.CaptureCallback ）

+ 可以在**ImageReader**设置的ImageReader.OnImageAvailableListener回调方法onImageAvailable(ImageReader reader)方法中获取拍照获得的临时照片，例如写入本地等。

+ 需要记得关闭CameraDevice。





**可以参考的文章**

[camera2的使用](https://juejin.im/post/5a33a5106fb9a04525782db5#heading-8)

[同样Android Things上无法同时拍照预览的情况](https://blog.csdn.net/liuweihhhh/article/details/61922708#commentBox)

