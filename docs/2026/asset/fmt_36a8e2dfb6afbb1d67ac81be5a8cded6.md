
![图片描述](https://bbs.kanxue.com/upload/attach/202004/757351_CAPVNY6P5QQBWBW.png)

我们再查看`Naver Mail_Mod.apk`的签名信息

此时的签名文件已经变成了我们自定义的签名文件`NETEASE.RSA`

![图片描述](https://bbs.kanxue.com/upload/attach/202004/757351_BFM3PU95KJ5DKEZ.png)

具体信息如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/757351_946EMW55CD99NZK.png)

然后我们安装自己签名的`Naver Mail_Mod.apk`依旧可以正常运行，说明该APK并没有进行签名验证。

### 解决方案

我们可以先确定好我们所使用的签名文件，然后通过`keytool`计算出该文件的MD5值，然后在代码中写一个验证，获取当前apk前面的MD5和预定义的MD5进行对比，如果不相同，则说明签名文件不是我们所指定的，说明文件被重新签名，这个时候可以终止运行。

为了测试方便，新建一个`Android studio`项目，然后通过代码获取当前的包名和签名文件的MD5，获取到的信息如下。我们可以看到，如果使用`Android studio`默认的签名MD5为:`90f2ba57841b2e30d56581f8d5c47c33`

![图片描述](https://bbs.kanxue.com/upload/attach/202004/757351_H8EGWR7YU3KQWVC.png)

于是我们将这个MD5硬编码到代码中进行比较。

![图片描述](https://bbs.kanxue.com/upload/attach/202004/757351_2B6DTRB4WXA3KGJ.png)

然后我们将生成的apk拿去反编译，然后再重新打包和签名。

![图片描述](https://bbs.kanxue.com/upload/attach/202004/757351_Y9GM8FUR4H9C3YQ.png)

然后我们安装`app_Mod.apk`，会发现运行直接闪退，因为检测到签名信息不一致。

这里更好的做法是将预定义的MD5值写到`JNI`层，这个后面再详细说说。

获取并判断签名的代码如下

```
package com.example.imageviewer.tool.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Context;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.pm.Signature;
import android.os.Bundle;
import android.util.Log;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        try {   
            PackageInfo packageInfo = getPackageManager().getPackageInfo(BuildConfig.APPLICATION_ID,
                    PackageManager.GET_SIGNATURES);
            String signValidString = getSignValidString(packageInfo.signatures[0].toByteArray());
            Log.e("get PackageName", BuildConfig.APPLICATION_ID );
            Log.e("get PackageSign",  signValidString);
            if(!signValidString.equals("90f2ba57841b2e30d56581f8d5c47c33"))
                finish();
        } catch (Exception e) {
            Log.e("get PackageName", "error:" + e);
        }
    }

    private String getSignValidString(byte[] paramArrayOfByte) throws NoSuchAlgorithmException {
        MessageDigest localMessageDigest = MessageDigest.getInstance("MD5");
        localMessageDigest.update(paramArrayOfByte);
        return toHexString(localMessageDigest.digest());
    }

    public String toHexString(byte[] paramArrayOfByte) {
        if (paramArrayOfByte == null) {
            return null;
        }
        StringBuilder localStringBuilder = new StringBuilder(2 * paramArrayOfByte.length);
        for (int i = 0; ; i++) {
            if (i >= paramArrayOfByte.length) {
                return localStringBuilder.toString();
            }
            String str = Integer.toString(0xFF & paramArrayOfByte[i], 16);
            if (str.length() == 1) {
                str = "0" + str;
            }
            localStringBuilder.append(str);
        }
    }
}
```

先占个坑

#### 0x02 应用权限测试

#### 0x03 系统备份测试

#### 0x04 调试检测

#### 0x05 四大组件暴露测试

#### 0x06 本地拒绝服务漏洞

#### 0x07 WebView密码明文存储

#### 0x08 日志泄露

#### 0x09 SharePreference数据安全测试

#### 0x0A 截屏测试

......

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
