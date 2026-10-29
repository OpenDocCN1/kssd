

```xml
<activity-alias android:name="com.android.camera.CameraLauncher" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.main.CameraActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    <meta-data android:name="android.app.shortcuts" android:resource="@xml/shortcuts"/>
</activity-alias>

<activity-alias android:name="com.android.camera.CameraActivity" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.main.CameraActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
    <meta-data android:name="com.android.keyguard.layout" android:resource="@layout/keyguard_widget"/>
</activity-alias>

<activity-alias android:label="@string/image_intent_label" android:name="com.android.camera.activity.CaptureActivity" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.CaptureActivity" android:visibleToInstantApps="true">
    <intent-filter>
        <action android:name="android.media.action.IMAGE_CAPTURE"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
    <meta-data android:name="com.android.keyguard.layout" android:resource="@layout/keyguard_widget"/>
</activity-alias>

<activity-alias android:label="@string/video_intent_label" android:name="com.android.camera.VideoCamera" android:screenOrientation="user" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.CaptureActivity" android:visibleToInstantApps="true">
    <intent-filter>
        <action android:name="android.media.action.VIDEO_CAPTURE"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity-alias>

<activity-alias android:name="com.android.camera.CameraImageActivity" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.CameraImageActivity" android:visibleToInstantApps="true">
    <intent-filter>
        <action android:name="android.media.action.STILL_IMAGE_CAMERA"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.VOICE"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.media.action.STILL_IMAGE_CAMERA"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="launch-camera" android:host="com.android.google.GoogleCamera" android:path="/still-image"/>
    </intent-filter>
    <meta-data android:name="com.android.keyguard.layout" android:resource="@layout/keyguard_widget"/>
</activity-alias>

<activity-alias android:name="com.android.camera.CameraVideoShortcutActivity" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.CameraImageActivity">
    <intent-filter>
        <action android:name="android.media.action.VIDEO_CAMERA"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.VOICE"/>
    </intent-filter>
    <meta-data android:name="com.android.keyguard.layout" android:resource="@layout/keyguard_widget"/>
</activity-alias>

<activity-alias android:name="com.android.camera.CameraDeepLinkActivity" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.CameraDeepLinkActivity" android:visibleToInstantApps="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:host="store.google.com" android:path="/product/pixelcamerateam1"/>
        <data android:scheme="http"/>
        <data android:scheme="https"/>
    </intent-filter>
    <meta-data android:name="com.android.keyguard.layout" android:resource="@layout/keyguard_widget"/>
</activity-alias>

<activity-alias android:name="com.android.camera.SecureCameraActivity" android:targetActivity="com.google.android.apps.camera.legacy.app.activity.SecureCameraActivity">
    <intent-filter>
        <action android:name="android.media.action.STILL_IMAGE_CAMERA_SECURE"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.VOICE"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.media.action.IMAGE_CAPTURE_SECURE"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
    <meta-data android:name="com.android.keyguard.layout" android:resource="@layout/keyguard_widget"/>
    <meta-data android:name="android.media.still_image_camera_preview_service" android:value="com.google.android.apps.camera.prewarm.NoOpPrewarmService"/>
</activity-alias>
```

发现对应的别名和 `targetActivity` 的对应关系是：

```text
com.android.camera.CameraLauncher -->              com.google.android.apps.camera.legacy.app.activity.main.CameraActivity

com.android.camera.CameraActivity -->              com.google.android.apps.camera.legacy.app.activity.main.CameraActivity

com.android.camera.activity.CaptureActivity -->    com.google.android.apps.camera.legacy.app.activity.CaptureActivity

com.android.camera.VideoCamera -->                 com.google.android.apps.camera.legacy.app.activity.CaptureActivity

com.android.camera.CameraImageActivity -->         com.google.android.apps.camera.legacy.app.activity.CameraImageActivity

com.android.camera.CameraVideoShortcutActivity --> com.google.android.apps.camera.legacy.app.activity.CameraImageActivity

com.android.camera.CameraDeepLinkActivity -->      com.google.android.apps.camera.legacy.app.activity.CameraDeepLinkActivity

com.android.camera.SecureCameraActivity -->        com.google.android.apps.camera.legacy.app.activity.SecureCameraActivity
```

## Activity 组件分析

加上之前无别名的组件，一共是以下 `activity` 组件。

上半部分无别名的组件因为直接 `android:exported="true"`，导致组件直接暴露，同时都没有设置 `android:permission`。下半部分的情况包括：`android:exported="true"` 直接暴露的组件，以及组件原来的 `exported` 属性是 `false`，但是 `alias` 别名对应的组件没有直接设置 `exported`，而是设置了 `intent-filter`。此时默认 `exported` 为 `true`，但使其响应的 `intent` 需符合 `intent-filter` 的所有条件。

```text
com.google.android.apps.camera.legacy.app.activity.main.CameraActivity

com.google.android.apps.camera.legacy.app.settings.CameraSettingsActivity

com.google.android.apps.camera.legacy.app.refocus.ViewerActivity

com.google.android.apps.camera.photobooth.activity.PhotoboothActivity

com.google.android.libraries.social.licenses.LicenseMenuActivity

---------------------------------------------------------------------

com.google.android.apps.camera.legacy.app.activity.main.CameraActivity

com.google.android.apps.camera.legacy.app.activity.CaptureActivity

com.google.android.apps.camera.legacy.app.activity.CameraImageActivity

com.google.android.apps.camera.legacy.app.activity.CameraDeepLinkActivity

com.google.android.apps.camera.legacy.app.activity.SecureCameraActivity
```

说实话，分析到这就感觉到这个漏洞原理应该还是比较简单的，因为它的组件暴露可以直接造成个人隐私泄露影响。具体功能逻辑也比较简单，就是有混淆，上 `jadx` 反混淆，遇到奇葩函数时，去 `jeb` 里看其 `smali` 源码，基本上就能摸清程序的逻辑了。

## 利用示例

其利用举个例子，比如这里的 `com.google.android.apps.camera.legacy.app.activity.CameraImageActivity`。我们已经知道 `com.google.android.apps.camera.legacy.app.activity.main.CameraActivity` 是主活动了，代码中的字符串已经暴露了它的用途，具体分析不再赘述。

而看 `CameraImageActivity` 发现它是直接调用 `CameraActivity` 的，所以说我们只要发出符合 `CameraImageActivity` 的 `intent`，那么就可以间接启动 `CameraActivity` 进行拍照等活动了。

- `CameraActivity`

  ![](https://bbs.kanxue.com/upload/attach/202002/837839_BWTV3E99D2G5CHA.png)

- `CameraImageActivity`

  ![](https://bbs.kanxue.com/upload/attach/202002/837839_X43F7X9CR45B4CK.png)

再看上面提到的官方描述时间线，能够体会到该漏洞等级之所以升级，也应该得益于 `checkmarx` 的 `exp`（即 `vuln app`）构造得好。

