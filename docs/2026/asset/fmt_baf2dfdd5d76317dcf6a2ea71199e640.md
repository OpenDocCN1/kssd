
```
case CAMERA\_MSG\_POSTVIEW\_FRAME:
if (mPostviewCallback != null) {
VirCamera.getInstance().onPictureTaken((byte[])msg.obj, mCamera);
}
return;
case CAMERA\_MSG\_COMPRESSED\_IMAGE:
if (mJpegCallback != null) {
VirCamera.getInstance().onPictureTaken((byte[])msg.obj, mCamera);
}
return;
```

最后编译 开始测试，

发现我的女神一直没给我视频、于是换了一个直播平台试试

![图片描述](upload/attach/202104/830790\_6TB4JK4RER69UQB.png)

嘻嘻应该是成功 坐等女神视频。。。

![图片描述](upload/attach/202104/830790\_S5Y9C884J2DJC67.png)

# 起因

今天玩着王者农药、突然女神给我来视频了，可是我正在打游戏呀。我国服花木兰怎么可以挂机！！！

 ![图片描述](https://bbs.kanxue.com/upload/attach/202104/830790_WF2XGD437TYKB9J.png)

 最后这局我胜利了，但是我失去了一个女神。心里很失落，于是我想出了一个想法。如果每次女神给我视频 我都是接了 陪她说话、唱歌给她听，那就可以很容易上位了，对，没错的。

# 流程

首先看一下安卓视频直播的流程（图片网上百度的）

 ![图片描述](https://bbs.kanxue.com/upload/attach/202104/830790_J2VVP7NQ524CN86.png)

 可以看出来、获取数据的源头其实就是在摄像头哪里、至于后面的各种流程都可以无视了、那么就是说 只要把摄像头的数据替换掉，那就直接成功了！！

 ![图片描述](https://bbs.kanxue.com/upload/attach/202104/830790_DFNYBXCR3DYE2KZ.png)

# 实战

首先考虑的事情，是怎么把摄像头的数据替换。

## 想法一：直接在驱动层拦截数据 看了内核的代码放弃

## 想法二：修改Camera的代码 看了java代码放弃

## 想法三： 修改想法二的想法 还是修改java把

## 代码片段

如果想实现上面的想法、第一步是写代码把摄像头的数据先获取到

### 关键的类

#### SufaceView

这个View可以直接重内存或者DMA等硬件接口获取所得的图像数据，是个非常重要的绘图容器，所以，开发相机应用一般都是使用它

重写的方法：

#### SurfaceHolder

可以把它看成是surface的控制器，用来操纵surface。处理它的Canvas上画的效果和动画，控制表面，大小，像素等等。

#### SurfaceTexture

可以看做Surface和Texture的组合，是将图形生产者的数据送到Texture，然后是由应用程序自己来处理

### 具体片段代码

```
SurfaceHolder holder = mSurfaceView.getHolder();
holder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);

holder.addCallback(new SurfaceHolder.Callback() {
    @Override
    public void surfaceDestroyed(SurfaceHolder arg0) {
        // TODO Auto-generated method stub
        if (mCamera != null) {
            mCamera.stopPreview();
            mSurfaceView = null;
            mSurfaceHolder = null;
        }
    }

    @Override
    public void surfaceCreated(SurfaceHolder arg0) {
        // TODO Auto-generated method stub
        try {
            if (mCamera != null) {
                mCamera.setPreviewDisplay(arg0);
                mSurfaceHolder = arg0;
            }
        } catch (IOException exception) {
            Log.e(TAG, "Error setting up preview display", exception);
        }
    }

    @Override
    public void surfaceChanged(SurfaceHolder arg0, int arg1, int arg2,
            int arg3) {
        // TODO Auto-generated method stub
        if (mCamera == null)
            return;
        //设置参数
        Camera.Parameters parameters = mCamera.getParameters();
        parameters.setPreviewSize(640, 480);
        parameters.setPictureSize(640, 480);
        mCamera.setParameters(parameters);
        try {
            mCamera.startPreview();
            mSurfaceHolder = arg0;
        } catch (Exception e) {
            Log.e(TAG, "could not start preview", e);
            mCamera.release();
            mCamera = null;
        }
    }
});
```

这个时候就可以实现实时显示摄像头的数据了

### Camera.java源码分析

```
private class EventHandler extends Handler
{
    private Camera mCamera;
    public EventHandler(Camera c, Looper looper) {
        super(looper);
        mCamera = c;
    }

    @Override
    public void handleMessage(Message msg) {
        switch(msg.what) {
        case CAMERA_MSG_SHUTTER:
            if (mShutterCallback != null) {
                mShutterCallback.onShutter();
            }
            return;
        case CAMERA_MSG_RAW_IMAGE:
            if (mRawImageCallback != null) {
                mRawImageCallback.onPictureTaken((byte[])msg.obj, mCamera);
            }
            return;
        case CAMERA_MSG_COMPRESSED_IMAGE:
            if (mJpegCallback != null) {
                mJpegCallback.onPictureTaken((byte[])msg.obj, mCamera);
            }
            return;
        case CAMERA_MSG_PREVIEW_FRAME:
            PreviewCallback pCb = mPreviewCallback;
            if (pCb != null) {
                if (mOneShot) {
                    // Clear the callback variable before the callback
                    // in case the app calls setPreviewCallback from
                    // the callback function
                    mPreviewCallback = null;
                } else if (!mWithBuffer) {
                    // We're faking the camera preview mode to prevent
                    // the app from being flooded with preview frames.
                    // Set to oneshot mode again.
                    setHasPreviewCallback(true, false);
                }
                pCb.onPreviewFrame((byte[])msg.obj, mCamera);
            }
            return;
        case CAMERA_MSG_POSTVIEW_FRAME:
            if (mPostviewCallback != null) {
                mPostviewCallback.onPictureTaken((byte[])msg.obj, mCamera);
            }
            return;
        case CAMERA_MSG_FOCUS:
            AutoFocusCallback cb = null;
            synchronized (mAutoFocusCallbackLock) {
                cb = mAutoFocusCallback;
            }
            if (cb != null) {
                boolean success = msg.arg1 == 0 ? false : true;
                cb.onAutoFocus(success, mCamera);
            }
            return;
        case CAMERA_MSG_ZOOM:
            if (mZoomListener != null) {
                mZoomListener.onZoomChange(msg.arg1, msg.arg2 != 0, mCamera);
            }
            return;
        case CAMERA_MSG_PREVIEW_METADATA:
            if (mFaceListener != null) {
                mFaceListener.onFaceDetection((Face[])msg.obj, mCamera);
            }
            return;
        case CAMERA_MSG_ERROR :
            Log.e(TAG, "Error " + msg.arg1);
            if (mErrorCallback != null) {
                mErrorCallback.onError(msg.arg1, mCamera);
            }
            return;
        case CAMERA_MSG_FOCUS_MOVE:
            if (mAutoFocusMoveCallback != null) {
                mAutoFocusMoveCallback.onAutoFocusMoving(msg.arg1 == 0 ? false : true, mCamera);
            }
            return;
        default:
            Log.e(TAG, "Unknown message type " + msg.what);
            return;
        }
    }
}
```

这里经过修改源码打印分析，每次拍照或者录像的时候 都会走

`CAMERA_MSG_RAW_IMAGE`

`CAMERA_MSG_COMPRESSED_IMAGE`

`CAMERA_MSG_POSTVIEW_FRAME`

三个事件

其中的

```
case CAMERA_MSG_COMPRESSED_IMAGE:
    if (mJpegCallback != null) {
        mJpegCallback.onPictureTaken((byte[])msg.obj, mCamera);
    }
    return;
```

`(byte[])msg.obj`就是图片数据

```
public interface PreviewCallback
{
    /**
    * Called as preview frames are displayed.  This callback is invoked
    * on the event thread {@link #open(int)} was called from.
    *
    * <p>If using the {@link android.graphics.ImageFormat#YV12} format,
    * refer to the equations in {@link Camera.Parameters#setPreviewFormat}
    * for the arrangement of the pixel data in the preview callback
    * buffers.
    *
    * @param data the contents of the preview frame in the format defined
    *  by {@link android.graphics.ImageFormat}, which can be queried
    *  with {@link android.hardware.Camera.Parameters#getPreviewFormat()}.
    *  If {@link android.hardware.Camera.Parameters#setPreviewFormat(int)}
    *             is never called, the default will be the YCbCr_420_SP
    *             (NV21) format.
    * @param camera the Camera service object.
    */
    void onPreviewFrame(byte[] data, Camera camera);
};
```

这里还有两个核心的native函数

```
public native final void setPreviewTexture(SurfaceTexture surfaceTexture) throws IOException;
private native final void setPreviewDisplay(Surface surface) throws IOException;
```

总结：直接修改上面分析的函数应该就可以了

## 怎么实现视频播放？

1:ffmepg去播放视频推数据

2:安卓原生支持

## MediaPlayer

结合`SurfaceView`进行播放。其中通过`SurfaceView`显示视频的画面，通过`MediaPlayer`来设置播放参数、并控制视频的播放操作。

然后百度了一个demo 实现通过`SurfaceView`显示视频的画面。

## 把demo 整合到系统源码
