
```
1 `c.a()` arguments: [0]`com.tencent.mm.protocal.protobuf.aaf@d4208e1` [1]false
2 `f.a()` arguments: [0]`com.tencent.mm.protocal.protobuf.aaf@d4208e1` [1][object Object] [2]false
3 `c.a()` arguments: [0]`com.tencent.mm.protocal.protobuf.aaf@d4208e1` [1][object Object] [2]false [3][object Object]
4 `c.processAddMsg()` arguments: [0]`AddMsgInfo(47744518)`, get[false], fault[false], up[false] fixTime[0] [1][object Object]
5 `b.b()` arguments: [0]`AddMsgInfo(47744518)`, get[false], fault[false], up[false] fixTime[0]
6 `ch.b()` arguments: [0]`AddMsgInfo(47744518)`, get[false], fault[false], up[false] fixTime[0]
7 `f.a()` arguments: [0]revokemsg [1][object Object] [2]`AddMsgInfo(47744518)`, get[false], fault[false], up[false] fixTime[0]
8 `handleRevokeMsgBySvrId` arguments: [0]796494781522958800
```

#### 4. `c.processAddMsg()`

发现是从`processAddMsg()`方法开始不一样的，也就是最先在这里判断是否为撤回消息的。

而判断是否进入第五层的消息是靠`v2 != null`来判断的，而`v2`来自`v3.vkP`，`v3`来自参数一`arg10.gpg`。

![image-20210816201709925](upload/attach/202108/925172_AVZQJP7TASH4TNS.jpg)

经调试，普通文本消息的`v3.vkP`为`1`，而撤回消息的`v3.vkP`为`10002`。

![image-20210816210123382](upload/attach/202108/925172_XAEQWKBNAUPWZVH.jpg)

而这恰好有个`10002`，不过并没有详细分析这里。

#### 5. `b.b()`

进入调用的`b(arg10)`方法，也就是`com.tencent.mm.s.b.b()`，没发现啥东西。

![image-20210816210228078](upload/attach/202108/925172_2YJR3GE6XA97THM.jpg)

**因为正常消息不会执行这个方法，当直接将该方法替换为空的时候，已经可以实现防撤回** :joy: 暴力 yyds，但是没有撤回提示，还得继续分析。

![image-20210816211759](upload/attach/202108/925172_52E2EDEKAHC2BHX.jpg)

#### 6. `ch.b()`

直接进入下一层`com.tencent.mm.model.ch.b`方法，在一开始又对`.vkP`做了一次判断。

![image-20210816210702683](upload/attach/202108/925172_HK39FSNSV7XMSGB.jpg)

至于`10001`暂不知作用。对于`10001`只有那一点处理，之后就`return null`了；而`default`分支最后也是`return null`，可以得知`.vkp`的含义是`msgType`。

可以知道这一个函数都是对撤回消息，也就是`msgType == 10002`进行处理的。

然后该方法下面都是对`v5`的判断逻辑，而`v5 = v0.GYI.IYz`。

将`v0.GYI` JSON 化输出，得到：

```json
// com.tencent.mm.protocal.protobuf.cv
{
  "CreateTime": 1629120000,
  "GYG": {
    "IYA": true,
    "IYz": "wxid_00000000000000",
    "includeUnKnownField": false
  },
  "GYH": {
    "IYA": true,
    "IYz": "wxid_00000000000000",
    "includeUnKnownField": false
  },
  "GYI": {
    "IYA": true,
    "IYz": "\u003csysmsg type\u003d\"revokemsg\"\u003e\n\t\u003crevokemsg\u003e\n\t\t\u003csession\u003ewxid_00000000000000\u003c/session\u003e\n\t\t\u003cmsgid\u003e1070000000\u003c/msgid\u003e\n\t\t\u003cnewmsgid\u003e600000000000000\u003c/newmsgid\u003e\n\t\t\u003creplacemsg\u003e\u003c![CDATA[\"Forgo7ten.\" 撤回了一条消息]]\u003e\u003c/replacemsg\u003e\n\t\u003c/revokemsg\u003e\n\u003c/sysmsg\u003e\n",
    "includeUnKnownField": false
  },
  "GYJ": 1,
  "GYK": {
    "hasBuffer": false,
    "hasILen": true,
    "iLen": 0,
    "includeUnKnownField": false
  },
  "GYN": 656800000,
  "nNf": 0,
  "vkP": 10002,
  "ysP": 1073900000,
  "ysR": 2498250000000000000,
  "data": [0, 0, 0, 0],
  "includeUnKnownField": false
}
```

而`v5`也就是：

```xml
<sysmsg type="revokemsg">
  <revokemsg>
    <session>wxid_##############</session>
    <msgid>10739#####</msgid>
    <newmsgid>7829338############</newmsgid>
    <replacemsg><![CDATA["Forgo7ten." 撤回了一条消息]]></replacemsg>
  </revokemsg>
</sysmsg>
```

尝试一下修改信息：

```javascript
function modify_revoke_str(old_revoke_str) {
  var message = "尝试撤回了一条消息";
  var revoke_msg = Java.use("java.lang.String").$new(message);
  var sub_str = Java.use("java.lang.String").$new("]]></replacemsg>");
  var index = old_revoke_str.indexOf(sub_str);
  var new_revoke_str =
    old_revoke_str.substring(0, index - 7) +
    revoke_msg +
    old_revoke_str.substring(index);
  return Java.use("java.lang.String").$new(new_revoke_str);
}
```

```javascript
Java.perform(function () {
  Java.openClassFile("/data/local/tmp/r0gson.dex").load();
  const gson = Java.use("com.r0ysue.gson.Gson");
  var clazz = Java.use("com.tencent.mm.model.ch");

  clazz.b.overload("com.tencent.mm.ak.e$a").implementation = function () {
    var log_str = "arguments:";
    for (var i = 0; i < arguments.length; i++) {
      log_str += " [" + i + "]" + arguments[i];
    }

    console.log("====================");

    var v0 = Java.cast(
      arguments[0].gpg.value,
      Java.use("com.tencent.mm.protocal.protobuf.cv")
    );
    console.log(gson.$new().toJson(v0));

    function modify_revoke_str(old_revoke_str) {
      var message = "尝试撤回了一条消息";
      var revoke_msg = Java.use("java.lang.String").$new(message);
      var sub_str = Java.use("java.lang.String").$new("]]></replacemsg>");
      var index = old_revoke_str.indexOf(sub_str);
      var new_revoke_str =
        old_revoke_str.substring(0, index - 7) +
        revoke_msg +
        old_revoke_str.substring(index);
      return Java.use("java.lang.String").$new(new_revoke_str);
    }

    var revoke_xml_obj = Java.cast(
      v0.GYI.value,
      Java.use("com.tencent.mm.protocal.protobuf.deo")
    );
    revoke_xml_obj.IYz.value = modify_revoke_str(revoke_xml_obj.IYz.value);

    console.log(revoke_xml_obj.IYz.value);
    console.log("====================");
    console.log("6 ch.b()", log_str);

    return clazz.b.overload("com.tencent.mm.ak.e$a").apply(this, arguments);
  };
});
```

虽然消息还是被撤回了，但也说明这种修改提示信息的方式是可行的。

![img](upload/attach/202108/925172_RVSZZVVZHAWY6EM.jpg)

通过 hook `notifyDataSetChanged`，JSON 打印最后一条消息发现：

撤回消息与撤回提示，仅有`field_content`（文本内容）、`field_type`、`exe`字段不一样。

其中正常消息的`field_type = 1, exe = true`；撤回提示`field_type = 10000, exe = false`。

被撤回消息和撤回消息的提示的`field_msgSvrId`字段是一样的，同时对应着撤回提示 XML 的`<newmsgid>`字段。

尝试修改撤回提示的`<newmsgid>`字段，发现没有什么用，直接就变成不撤回了。

### 返回该函数的分析

大概是对 XML 进行分析处理的一些东西。

![image-20210816235316259](upload/attach/202108/925172_33T3C4AF6DGZVWS.jpg)

之后就去了`com.tencent.mm.model.f.a()`方法：

```java
public final com.tencent.mm.ak.e.b a(
    String arg24,
    Map arg25,
    com.tencent.mm.ak.e.a arg26
);
```

- 参数一为字符串 `String`，获取过程为上图中蓝框。
- 即获取的是 XML 中`sysmsg`的`type`属性。
- 撤回消息就为`"revokemsg"`。
- 参数二是一个 XML 内容转成的 Map 数组：

```json
{
  ".sysmsg.revokemsg.newmsgid": "4308###############",
  ".sysmsg.revokemsg": "\n\t",
  ".sysmsg.revokemsg.session": "wxid_##############",
  ".sysmsg": "\n",
  ".sysmsg.$type": "revokemsg",
  ".sysmsg.revokemsg.replacemsg": "\"Forgo7ten.\" 撤回了一条消息",
  ".sysmsg.revokemsg.msgid": "##########"
}
```

- 参数三保存着原始的所有数据：

```java
// com.tencent.mm.ak.e$a
{
  "gpg": {
    "CreateTime": 1629120000,
    "GYG": {
      "IYA": true,
      "IYz": "wxid_00000000000000",
      "includeUnKnownField": false
    },
    "GYH": {
      "IYA": true,
      "IYz": "wxid_00000000000000",
      "includeUnKnownField": false
    },
    "GYI": {
      "IYA": true,
      "IYz": "\u003csysmsg type\u003d\"revokemsg\"\u003e\n\t\u003crevokemsg\u003e\n\t\t\u003csession\u003ewxid_00000000000000\u003c/session\u003e\n\t\t\u003cmsgid\u003e1070000000\u003c/msgid\u003e\n\t\t\u003cnewmsgid\u003e600000000000000\u003c/newmsgid\u003e\n\t\t\u003creplacemsg\u003e\u003c![CDATA[\"Forgo7ten.\" 撤回了一条消息]]\u003e\u003c/replacemsg\u003e\n\t\t\u003c/revokemsg\u003e\n\u003c/sysmsg\u003e\n",
      "includeUnKnownField": false
    },
    "GYJ": 1,
    "GYK": {
      "hasBuffer": false,
      "hasILen": true,
      "iLen": 0,
      "includeUnKnownField": false
    },
    "GYN": 656800000,
    "nNf": 0,
    "vkP": 10002,
    "ysP": 1073900000,
    "ysR": 2498250000000000000,
    "data": [0, 0, 0, 0]
  },
  "hQb": false,
  "hQc": false,
  "hQd": false,
  "hQe": 0,
  "hQf": false,
  "what": 0
}
```

#### 7. `f.a()`

这个方法里面是分块对参数一，也就是`.sysmsg.$type`的值进行判断。

![image-20210817135838492](upload/attach/202108/925172_XMCMQ4P4PN4HJMJ.jpg)

之后发现将`.a()`方法置为空则消息不会撤回，继续跟入。

![image-20210817140049459](upload/attach/202108/925172_ZQC7MU47MCSTP8V.jpg)

在里面发现了数据库操作方法 `update`。

![image-20210817140125502](upload/attach/202108/925172_N89V6F8NQ77JSWM.jpg)

之后嵌套了三个`update`，最后来到了`updateWithOnConflict`方法。

![image-20210817142313024](upload/attach/202108/925172_2TXMXRFF3FQHAA3.jpg)

附上堆栈信息。
