
`android.os.Debug.isDebuggerConnected()`；判断当前应用有没有被调试

```java
/*
* 执行 timeout()
* 注意：如果处于程序调试状态和CC.DEBUG是true，
* 两个都满足情况下，不执行超时 timeout()
* @param cc
*/
private void executeTimeout(CC cc) {
if (!CC.DEBUG) {
cc.timeout();
return;
}
if (!Debug.isDebuggerConnected()) {
cc.timeout();
}
}
```

### 获取 debuggable 值的API接口

```java
void detectOsDebug(){
boolean connected = android.os.Debug.isDebuggerConnected();
Log.d(TAG, "debugger connect status:" + connected);
}
```

### 电话监听

```java
public class PreciseCallState implements Parcelable {
public static final int PRECISE_CALL_STATE_IDLE = 0;
// 通话空闲
public static final int PRECISE_CALL_STATE_ACTIVE = 1;
// 正在通话
public static final int PRECISE_CALL_STATE_HOLDING = 2;
// 通话挂起
public static final int PRECISE_CALL_STATE_DIALING = 3;
// 拨号开始
public static final int PRECISE_CALL_STATE_ALERTING = 4;
// 正在呼出
public static final int PRECISE_CALL_STATE_INCOMING = 5;
// 对方来电
public static final int PRECISE_CALL_STATE_WAITING = 6;
// 第三方来电等待
public static final int PRECISE_CALL_STATE_DISCONNECTED = 7;
// 挂断完成
public static final int PRECISE_CALL_STATE_DISCONNECTING = 8;
// 正在挂断
```

### 获取手机号码

```java
static public String getPhone(Context p0){
String str = "";
try{
return p0.getSystemService("phone").getLine1Number().replace("+86", str).replaceAll(" ", str);
}catch(java.lang.Exception e-1){
return str;
}
}
```

### 获取当前位置

```java
static public Location c(Context p0){
LocationManager lSystemServi;
Object[] objectArray;
String sBestProvide;
Object[] objectArray1;
Object[] objectArray2;
int vi = 1;
try{
if ((lSystemServi = p0.getSystemService("location"))) {
Criteria criteria = new Criteria();
criteria.setPowerRequirement(vi);
criteria.setAccuracy(2);
if ((sBestProvide = lSystemServi.getBestProvider(criteria, vi))) {
Location lLastKnownLo = lSystemServi.getLastKnownLocation(sBestProvide);
objectArray1 = new Object[vi];
objectArray1[0]=lLastKnownLo;
c.b(e.a, "Location found: %s", objectArray1);
return lLastKnownLo;
}else {
objectArray2 = new Object[0];
c.a(e.a, "Location Manager provider is null.", objectArray2);
}
}else {
objectArray2 = new Object[0];
c.a(e.a, "Location Manager is null.", objectArray2);
}
}catch(java.lang.Exception e5){
objectArray = new Object[vi];
objectArray[0]=e5.toString();
c.a(e.a, "Failed to retrieve location: %s", objectArray);
}
return null;
}
```

## Android 中的危险权限

![图片描述](https://bbs.kanxue.com/upload/attach/202104/767964_J8K2V7CE48XJRJ6.png)

### 参考资料

- 《Android API 参考文档》
- 《GDA 官方资料》
- 《Android 取证资料》
- 《Android 权限调用相关资料》
- 《Android 中的危险权限表格》

## 未完待续

### 当前更新日期：2023年5月17日

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-8-25 11:22

被梦幻的彼岸编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)

[#其他](https://bbs.kanxue.com/forum-161-1-129.htm)
