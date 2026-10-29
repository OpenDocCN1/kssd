
那么现在就是见神奇的时候

```java
Camera(int cameraId) {
    mShutterCallback = null;
    mRawImageCallback = null;
    mJpegCallback = null;
    mPreviewCallback = null;
    mPostviewCallback = null;
    mUsingPreviewAllocation = false;
    mZoomListener = null;
    Looper looper;

    if ((looper = Looper.myLooper()) != null) {
        mEventHandler = new EventHandler(this, looper);
    } else if ((looper = Looper.getMainLooper()) != null) {
        mEventHandler = new EventHandler(this, looper);
    } else {
        mEventHandler = null;
    }

    String packageName = ActivityThread.currentPackageName();
    native_setup(new WeakReference<Camera>(this), cameraId, packageName);
    // 开启虚拟摄像头
    VirCamera.getInstance().open();
}
```

```java
case CAMERA_MSG_POSTVIEW_FRAME:
    if (mPostviewCallback != null) {
        VirCamera.getInstance().onPictureTaken((byte[])msg.obj, mCamera);
    }
    return;

case CAMERA_MSG_COMPRESSED_IMAGE:
    if (mJpegCallback != null) {
        VirCamera.getInstance().onPictureTaken((byte[])msg.obj, mCamera);
    }
    return;
```

最后编译 开始测试，发现我的女神一直没给我视频、于是换了一个直播平台试试

![图片描述](https://bbs.kanxue.com/upload/attach/202104/830790_6TB4JK4RER69UQB.png)

嘻嘻应该是成功 坐等女神视频。。。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/830790_S5Y9C884J2DJC67.png)

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)

[#程序开发](https://bbs.kanxue.com/forum-161-1-124.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
