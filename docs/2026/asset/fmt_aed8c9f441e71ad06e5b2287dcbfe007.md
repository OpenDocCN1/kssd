
# 旅行青蛙存档修改

## 代码实现

```java
package com.example.frog;

import android.content.Context;
import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.view.inputmethod.InputMethodManager;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;

public class MainActivity extends AppCompatActivity {
    private EditText editText;
    private EditText editText2;
    private Button button;
    private InputMethodManager inputMethodManager;
    private static final String FILE_PATH = Environment.getExternalStorageDirectory() + File.separator + "Android/data/jp.co.hit_point.tabikaeru/files/Tabikaeru.sav";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        editText = (EditText) findViewById(R.id.editText);
        editText2 = (EditText) findViewById(R.id.editText2);
        button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (editText.getText().toString().equals("") || editText2.getText().toString().equals("")) {
                    return;
                }
                String cloverHex = String.format("%06X",  Integer.valueOf(editText.getText().toString()));
                String couponHex = String.format("%06X",  Integer.valueOf(editText2.getText().toString()));
                Log.d("123", " " + cloverHex);
                Log.d("123", " " + couponHex);
                writeToFile(cloverHex, couponHex);
            }
        });
    }

    public void writeToFile(String cloverHex, String couponHex) {
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;
        File file = new File(FILE_PATH);
        File newFile = new File(FILE_PATH);
        byte[] cloverByteArray = hexStringToByte(cloverHex);
        byte[] couponByteArray = hexStringToByte(couponHex);
        if (!file.exists()) {
            Log.d("123", "未找到文件Tabikaeru.sav");
            return;
        }
        try {
            fileInputStream = new FileInputStream(file);
            byte[] arrayOfByte = new byte[fileInputStream.available()];
            Log.d("123", "文件大小" + arrayOfByte.length);
            fileInputStream.read(arrayOfByte);
            if (arrayOfByte.length > 29) {
                file.delete();
                Log.d("123", "删除旧文件");
                createFile(newFile);
                //三叶草
                arrayOfByte[23] = cloverByteArray[0];
                arrayOfByte[24] = cloverByteArray[1];
                arrayOfByte[25] = cloverByteArray[2];
                //抽奖券
                arrayOfByte[27] = couponByteArray[0];
                arrayOfByte[28] = couponByteArray[1];
                arrayOfByte[29] = couponByteArray[2];
                Log.d("123", " " + arrayOfByte.length);
                for (int i = 0; i <arrayOfByte.length; i++) {
                    Log.d("123", " " + arrayOfByte[i]);
                }
                fileOutputStream = new FileOutputStream(newFile);
                fileOutputStream.write(arrayOfByte);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            Toast.makeText(this, getString(R.string.saved), Toast.LENGTH_SHORT).show();
            hideSoftInput();
            try {
                if (fileInputStream != null) {
                    fileInputStream.close();
                }
                if (fileOutputStream != null) {
                    fileOutputStream.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public void createFile(File file){
        try{
            file.getParentFile().mkdirs();
            file.createNewFile();
        }catch (IOException e){
            e.printStackTrace();
        }
    }

    public void hideSoftInput(){
        if(inputMethodManager == null) {
            inputMethodManager = (InputMethodManager)this.getSystemService(Context.INPUT_METHOD_SERVICE);
        }
        inputMethodManager.hideSoftInputFromWindow(editText.getWindowToken(), 0);
        editText.clearFocus();
        inputMethodManager.hideSoftInputFromWindow(editText2.getWindowToken(), 0);
        editText2.clearFocus();
    }

    /**
     * 把16进制字符串转换成字节数组
     */
    public static byte[] hexStringToByte(String hex) {
        int len = (hex.length() / 2);
        byte[] result = new byte[len];
        char[] achar = hex.toCharArray();
        for (int i = 0; i < len; i++) {
            int pos = i * 2;
            result[i] = (byte) (toByte(achar[pos]) << 4 | toByte(achar[pos + 1]));
            if (result[i] == 0) {
                result[i] = 00;
            }
        }
        return result;
    }

    private static int toByte(char c) {
        byte b = (byte) "0123456789ABCDEF".indexOf(c);
        return b;
    }
}
```

上述代码实现了存档的直接修改，界面如下，不需要ROOT权限：

![](https://bbs.kanxue.com/upload/attach/201801/654083_ESWJEMDKSEDKY58.jpg)

输入数值后，点击修改即可完成三叶草及抽奖券的修改，更多物品修改请自行尝试。

## 0x3. apk修改

思路：分析`apk`包，找到脚本文件，反编译后找到关键`method`进行修改，然后重新打包

工具：`Android Killer`，`DnSpy`

`Android Killer`相关操作这里不再赘述，反编译后我们发现这是一个`mono`框架的`Unity3D`游戏，`Unity3D`游戏的脚本文件都存放在`Assembly-CSharp.dll`或`Assembly-CSharp-firstpass.dll`文件中，很显然，旅行青蛙的脚本文件位于`Assembly-CSharp.dll`，我们使用`Dnspy`进行分析看看。

![](https://bbs.kanxue.com/upload/attach/201801/654083_XMC236MREK9W94X.jpg)

我们搜索三叶草的英文`clover`，发现`getCloverPoint`可能是我们需要找的关键`method`。

![](https://bbs.kanxue.com/upload/attach/201801/654083_YERA6M69KKS8GHC.jpg)

根据`getCloverPoint`的源码，我们发现这个`method`的功能是在三叶草数量发生变化时在三叶草数量进行增减运算，那么我们可以对函数内部增加数量的这句代码进行修改，修改为发生变化增加固定数量的三叶草，比如`10000`。

抽奖券相关修改也在`SuperGameMaster`中可以找到，`method`名为`getTicket`，此处不作演示，请大家自己尝试修改

![](https://bbs.kanxue.com/upload/attach/201801/654083_C99EK96S34MB5U9.jpg)

修改后函数变更为：

![](https://bbs.kanxue.com/upload/attach/201801/654083_ZDKGQFZ5DWD836R.jpg)

保存后打包`apk`运行，只要三叶草数量发生变化（如收割三叶草或者购买物品），三叶草的数量就会增加`10000`。

## 0x4. 总结

本文通过多种思路对旅行青蛙的修改进行了分析，内容较为简单，主要目的是分享游戏破解分析的思路，有兴趣的可以尝试更多物品数量的修改。

![](https://bbs.kanxue.com/upload/attach/201901/93_AVDGGRCARXQHB9A.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-1-11 19:33 被kanxue编辑，原因：

上传的附件：

- `app-debug.apk`
  （1.48MB，612次下载）
- `Frog_源码.zip`
  （1.08MB，285次下载）
