
# 通过2015年AliCrackMe学习Android逆向（一）

总感觉以前学的 Android 逆向不扎实，现在通过这五道题复习一下。

## AliCrackme\_1

### 1. 运行题目

> MuMu模拟器

需要输入正确的密码。

![img](upload/attach/202107/830671\_XJQZ6YYJUWZ7YT8.jpg)

### 2. 反编译分析

首先对apk文件进行反编译，找到登录按钮所在Activity（`MainActivity`）的代码并分析。

![img](upload/attach/202107/830671\_T8BHCA29X5DQNSS.jpg)

- `table` 是使用方法 `getTableFromPic` 得到的一张字符表;
- `pw` 是使用方法 `getPwdFromPic` 通过 `table` 进行编码后的正确密码
- `enPassword` 是对在输入框中输入的字符串进行编码的结果。

### 3. 泄漏 `table` 和 `pw`

#### 方法一：`ddms`/`adb logcat`

它们都会通过`Log`打印出来，如上图中红框所示。所以，我们可以利用`DDMS`或者`adb logcat`查看log。通过在APP中随便输入字符串，点击“登录”按钮，就能从Logcat中查看到打印值。如下图所示，输入“123456”。

![img](upload/attach/202107/830671\_Q9EU9B4KYKUKCUQ.jpg)

后面写一个脚本，使用`MainActivity`中本身提供的解码方法，传入参数`table`和`pw`，就能得到正确密码。

#### 方法二：逆向分析

如果没有`Log.d`方法，该怎么办呢？

前面已经分析出`table`和`pw`分别是通过方法`getTableFromPic`和`getPwdFromPic`得到的，那么就分析这两个方法。它们都是通过对`assets`目录下的`logo.png`图片进行计算得到返回值。那么就可以创建一个Android项目，把`logo.png`拷贝到新项目的`assets`目录下。然后将`getTableFromPic`和`getPwdFromPic`拷贝到`AndroidTest`目录下的`ExampleInstrumentedTest`文件中。

![img](upload/attach/202107/830671\_Z5C3WJB5GE6V94N.jpg)

```java
package com.lzx.ufo;
import android.content.Context;
import android.util.Log;
import androidx.test.platform.app.InstrumentationRegistry;
import androidx.test.ext.junit.runners.AndroidJUnit4;
import org.junit.Test;
import org.junit.runner.RunWith;
import java.io.IOException;
import java.io.InputStream;
import static org.junit.Assert.\*;
/\*\*
\* Instrumented test, which will execute on an Android device.
\*
\* @see <a href="http://d.android.com/tools/testing">Testing documentation</a>
\*/
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
@Test
public void useAppContext() {
// Context of the app under test.
Context appContext = InstrumentationRegistry.getInstrumentation().getTargetContext();
assertEquals("com.lzx.ufo", appContext.getPackageName());
Log.d("UFO======",getTableFromPic(appContext));
Log.d("UFO======",getPwdFromPic(appContext));
}
protected String getTableFromPic(Context appContext) {
InputStream is = null;
String value = "";
try {
is = appContext.getResources().getAssets().open("logo.png");
int lenght = is.available();
// Log.d("UFO","Table-lenght:"+lenght);
byte[] b = new byte[lenght];
is.read(b, 0, lenght);
byte[] data = new byte[768];
System.arraycopy(b, 89473, data, 0, 768);
String value2 = new String(data, "utf-8");
if (is == null) {
return value2;
}
try {
is.close();
return value2;
} catch (IOException e) {
return value2;
}
} catch (Exception e2) {
e2.printStackTrace();
if (is == null) {
return value;
}
try {
is.close();
return value;
} catch (IOException e3) {
return value;
}
} catch (Throwable th) {
if (is != null) {
try {
is.close();
} catch (IOException e4) {
}
}
}
return value;
}
protected String getPwdFromPic(Context appContext) {
InputStream is = null;
String value = "";
try {
is = appContext.getResources().getAssets().open("logo.png");
int lenght = is.available();
// Log.d("UFO","Pwd-lenght:"+lenght);
byte[] b = new byte[lenght];
is.read(b, 0, lenght);
byte[] data = new byte[18];
System.arraycopy(b, 91265, data, 0, 18);
String value2 = new String(data, "utf-8");
if (is == null) {
return value2;
}
try {
is.close();
return value2;
} catch (IOException e) {
return value2;
}
} catch (Exception e2) {
e2.printStackTrace();
if (is == null) {
return value;
}
try {
is.close();
return value;
} catch (IOException e3) {
return value;
}
} catch (Throwable th) {
if (is != null) {
try {
is.close();
} catch (IOException e4) {
}
}
}
return value;
}
}
```

![img](upload/attach/202107/830671\_ENC2CMVEKAC7XGA.jpg)

### 4. 编写脚本计算flag

#### 方法一：拷贝题目中的解码方法，使用Java代码解题

```java
public class Answer{
public static void main(String[] args){
String table = "一乙二十丁厂七卜人入八九几儿了力乃刀又三于干亏士工土才寸下大丈与万上小口巾山千乞川亿个勺久凡及夕丸么广亡门义之尸弓己已子卫也女飞刃习叉马乡丰王井开夫天无元专云扎艺木五支厅不太犬区历尤友匹车巨牙屯比互切瓦止少日中冈贝内水见午牛手毛气升长仁什片仆化仇币仍仅斤爪反介父从今凶分乏公仓月氏勿欠风丹匀乌凤勾文六方火为斗忆订计户认心尺引丑巴孔队办以允予劝双书幻玉刊示末未击打巧正扑扒功扔去甘世古节本术可丙左厉右石布龙平灭轧东卡北占业旧帅归且旦目叶甲申叮电号田由史只央兄叼叫另叨叹四生失禾丘付仗代仙们仪白仔他斥瓜乎丛令用甩印乐";
String pwd = "义弓么丸广之";
System.out.println(aliCodeToBytes(table,pwd));
}
private static String aliCodeToBytes(String codeTable, String strCmd) {
StringBuilder sb = new StringBuilder();
byte[] cmdBuffer = new byte[strCmd.length()];
for (int i = 0; i < strCmd.length(); i++) {
cmdBuffer[i] = (byte) codeTable.indexOf(strCmd.charAt(i));
sb.append((char)cmdBuffer[i]);
}
return sb.toString();
}
}
```

![img](upload/attach/202107/830671\_ZKPKWA24XWBTYYE.jpg)

#### 方法二：根据解码方法，编写对应的Python脚本

```python
table = [
'一', '乙', '二', '十', '丁', '厂', '七', '卜', '人', '入', '八', '九', '几', '儿', '了', '力',
'乃', '刀', '又', '三', '于', '干', '亏', '士', '工', '土', '才', '寸', '下', '大', '丈', '与',
'万', '上', '小', '口', '巾', '山', '千', '乞', '川', '亿', '个', '勺', '久', '凡', '及', '夕',
'丸', '么', '广', '亡', '门', '义', '之', '尸', '弓', '己', '已', '子', '卫', '也', '女', '飞',
'刃', '习', '叉', '马', '乡', '丰', '王', '井', '开', '夫', '天', '无', '元', '专', '云', '扎',
'艺', '木', '五', '支', '厅', '不', '太', '犬', '区', '历', '尤', '友', '匹', '车', '巨', '牙',
'屯', '比', '互', '切', '瓦', '止', '少', '日', '中', '冈', '贝', '内', '水', '见', '午', '牛',
'手', '毛', '气', '升', '长', '仁', '什', '片', '仆', '化', '仇', '币', '仍', '仅', '斤', '爪',
'反', '介', '父', '从', '今', '凶', '分', '乏', '公', '仓', '月', '氏', '勿', '欠', '风', '丹',
'匀', '乌', '凤', '勾', '文', '六', '方', '火', '为', '斗', '忆', '订', '计', '户', '认', '心',
'尺', '引', '丑', '巴', '孔', '队', '办', '以', '允', '予', '劝', '双', '书', '幻', '玉', '刊',
'示', '末', '未', '击', '打', '巧', '正', '扑', '扒', '功', '扔', '去', '甘', '世', '古', '节',
'本', '术', '可', '丙', '左', '厉', '右', '石', '布', '龙', '平', '灭', '轧', '东', '卡', '北',
'占', '业', '旧', '帅', '归', '且', '旦', '目', '叶', '甲', '申', '叮', '电', '号', '田', '由',
'史', '只', '央', '兄', '叼', '叫', '另', '叨', '叹', '四', '生', '失', '禾', '丘', '付', '仗',
'代', '仙', '们', '仪', '白', '仔', '他', '斥', '瓜', '乎', '丛', '令', '用', '甩', '印', '乐'
]
pw= ['义','弓','么','丸','广','之']
flag = ''
pw\_len = 0
print(len(table))
print(len(pw))
for i in range(len(pw)):
for j in range(len(table)):
if table[j] == pw[i]:
flag += chr(j)
break
print(flag)
```

![img](upload/attach/202107/830671\_7T43TBQ99ERJ3GD.jpg)

## AliCrackme\_2

### 1. 运行题目

> Nexus 5 , Android 4.4.4

![image-20210726123525520](upload/attach/202107/830671\_2MS847R8PGCFBYE.jpg)

### 2. 反编译分析

`jadx`反编译，发现校验方法`securityCheck`是一个native方法。

![img](upload/attach/202107/830671\_P3YEK8VW83MJMQH.jpg)

于是，使用`ida`打开文件`libcrackme.so`，找到`securityCheck`对应的JNI函数`Java_com_yaotong_crackme_MainActivity_securityCheck`。修改一些变量名和变量类型（`a1`的类型修改为`JNIEnv*`）。

![img](upload/attach/202107/830671\_QDV6B5AW38BQB9S.jpg)

从返回值往前分析，我们需要返回true，也就是1，那就需要输入的字符串和`off_628C`存储的字符串相等。

![img](upload/attach/202107/830671\_39HF6G2QCXZUVCJ.jpg)

![img](upload/attach/202107/830671\_JVDSSQZZEX4CJSF.jpg)

将`wojiushidaan`输入，显示输入密码错误。因此，可以猜测某个地方在执行`while`循环之前修改了变量`aWojiushidaan`的值。

到这，目的就很明确了，找到变量`aWojiushidaan`变换后的值。

### 3. 泄漏`aWojiushidaan`变换后的值，得到flag

#### 方法一：`frida` hook
