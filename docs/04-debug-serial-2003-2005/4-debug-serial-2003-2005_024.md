
# 西门子手机短信通

> 原文：[`www.pediy.com/kssd/pediy06/pediy6039.htm`](https://www.pediy.com/kssd/pediy06/pediy6039.htm)

破解目标：西门子手机短信通

破解工具：`trw2000`, `UltraEdit`

日期：2003.5.4

破解难度：中

下载地址：`http://hnesd.com/rj/xmz.rar`

软件功能：

必备硬件：

1. 软件安装光盘
2. 西门子手机连接线
3. 西门子 3518/3508 等手机
4. 电脑（需要电脑至少有一个串口，用于连接手机连接线）

## 一、单发

点击“功能选择”下的“单发”，可以给某一个手机发送短信！

## 二、连号呼

用于给某一段手机发送短信。

点击“功能选择”下的“连号呼”，在弹出窗口中输入要发送的开始手机号和结束手机号，再输入信息内容，再点击“确定”，程序将开始生成发送数据，根据要发送手机的数量不同生成数据时间有一点差异，但不影响发送速度！

## 三、群发

用于给特定一群人发送短信，例如：某一公司或某一公司的某一个部门。

点击“功能选择”下的“群呼”，在弹出的窗口中可以看到一个类似“资源管理器”的界面，双击“组名”，展开树形结构，再点击菜单“分组管理”，可以增加组，如公司部门，如果要删除或更改某一个组，请先选择（即点击）这个组，再点击菜单“分组管理”，就可以看到“删除组”和“更改组”都可以使用了。

如果是增加用户到某一个组，也请先选择这个组，再点击菜单“用户管理”。

## 四、注册

正式版在第一次启用需提供本地 `ID` 号后的数字给软件商再根据软件商提供的注册号进行注册。

（正式版需注册才可使用！）

### 开工

打开西门子手机短信通软件，在注册窗口中输入`77887788`8 位注册码，打开`trw2000`，下`bpx hmemcpy`，点注册，程序被中断。再下`pmodule`

一路`F10`来到:

```
* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
                          |
:00441091 FF15EC114000           Call dword ptr [004011EC]
:00441097 8B45E0                 mov eax, dword ptr [ebp-20]
:0044109A 50                     push eax   <-----输入的注册码“77887788”

* Reference To: MSVBVM60.__vbaI4Str, Ord:0000h
                          |
:0044109B 8B3D84114000           mov edi, dword ptr [00401184]
:004410A1 FFD7                   call edi
:004410A3 8BD0                   mov edx, eax

* Possible StringData Ref from Code Obj ->"14201496"
                          |
:004410A5 6824834000             push 00408324       <----内部密钥“14201496”
:004410AA 899514FFFFFF          mov dword ptr [ebp+FFFFFF14], edx
:004410B0 FFD7                   call edi
:004410B2 8B8D14FFFFFF          mov ecx, dword ptr [ebp+FFFFFF14]
:004410B8 33C8                   xor ecx, eax
:004410BA 51                     push ecx

* Reference To: MSVBVM60.__vbaStrI4, Ord:0000h
                          |
:004410BB FF1520104000           Call dword ptr [00401020]  <--计算关键 call（调用 code.dll)
:004410C1 8BD0                   mov edx, eax
:004410C3 8D4DE0                 lea ecx, dword ptr [ebp-20]
:004410C6 FFD3                   call ebx
:004410C8 8B55E0                 mov edx, dword ptr [ebp-20]
:004410CB 52                     push edx       <----d eax (看到”77887788“与“14201496”
                                                经计算后的数值）

-------------------------------------------------------------------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00441114(C)
|
:00441128 8B55DC                 mov edx, dword ptr [ebp-24]
:0044112B 52                     push edx       <----d edx (看到机器码”)
:0044112C 8B45E0                 mov eax, dword ptr [ebp-20]
:0044112F 50                     push eax   <----d eax (看到”77887788“与“14201496”
                                   经计算后的数值“75287476”)
                                   如果 edx=eax 则注册成功。
```

好家伙！用输入的注册码经计算后与机器码比较，没头绪啦！用`UltraEdit`改内部密钥`14201496`为`00000000`看看，（查找`31003400320030003100340039003600`改为`30003000300030003000300030003000`）

重新用`trw2000`加载，输入注册码`83291823`跟机器码相等，又来到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00441114(C)
|
:00441128 8B55DC                 mov edx, dword ptr [ebp-24]
:0044112B 52                     push edx       <----d edx (看到机器码”83291823”)
:0044112C 8B45E0                 mov eax, dword ptr [ebp-20]
:0044112F 50                     push eax   <----d eax (看到”83291823“与“00000000”
                                   经计算后的数值“38928132”)
                                   如果 edx=eax 则注册成功。
```

好，看到没有---经计算后的数值`38928132`为输入的注册码两位两位的反转得来的

如果我输入注册码`38928132`就会生成`83291823`注册就成功！！！

### 总结

用`UltraEdit`改内部密钥`14201496`为`00000000`看看，（查找`31003400320030003100340039003600`改为`30003000300030003000300030003000`）注册码即为机器码两位两位的反转得来。

johnroot
2003.5.4 下午 于长沙

---

# 标题: 算法

*   作者：sunboy

详细信息:

```
* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
                          |
:00441091 FF15EC114000           Call dword ptr [004011EC]
:00441097 8B45E0                 mov eax, dword ptr [ebp-20]
:0044109A 50                     push eax   <-----输入的注册码“12345678”
                                             已经变换为 21436587

* Reference To: MSVBVM60.__vbaI4Str, Ord:0000h
                          |
:0044109B 8B3D84114000           mov edi, dword ptr [00401184]
:004410A1 FFD7                   call edi
:004410A3 8BD0                   mov edx, eax

* Possible StringData Ref from Code Obj ->"14201496"
                          |
:004410A5 6824834000             push 00408324       <----内部密钥“14201496”
:004410AA 899514FFFFFF          mov dword ptr [ebp+FFFFFF14], edx
:004410B0 FFD7                   call edi
:004410B2 8B8D14FFFFFF          mov ecx, dword ptr [ebp+FFFFFF14] //10 进制 21436587

:004410B8 33C8                   xor ecx, eax     //实际上在这里计算
      ecx=14718AB= 10 进制 21436587(输入的注册码)  eax=D8B298= 10 进制 14201496(内部密钥)
      经过异或运算后 ecx=19FAA33 十进制为 27241011

:004410BA 51                     push ecx

* Reference To: MSVBVM60.__vbaStrI4, Ord:0000h
                          |
:004410BB FF1520104000           Call dword ptr [00401020]  <--计算关键 call（调用 code.dll)
                          此 call 后 eax=27241011

:004410C1 8BD0                   mov edx, eax
:004410C3 8D4DE0                 lea ecx, dword ptr [ebp-20]
:004410C6 FFD3                   call ebx
:004410C8 8B55E0                 mov edx, dword ptr [ebp-20]
:004410CB 52                     push edx       <----d eax (看到”77887788“与“14201496”
                                             经计算后的数值）

---------------------------[ebp-20]存放的是计算结果即 27241011 -------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00441114(C)
|
:00441128 8B55DC                 mov edx, dword ptr [ebp-24]
:0044112B 52                     push edx       <----d edx (看到机器码”)
:0044112C 8B45E0                 mov eax, dword ptr [ebp-20]
:0044112F 50                     push eax   <----d eax (看到”77887788“与“14201496”
                                   经计算后的数值“75287476”)
                                   如果 edx=eax 则注册成功。
```

你分析出如果 `edx=eax` 则注册成功，那么由上面的算法反推就可以知道真正的注册码了。

不足之处请大家指点！

`4dd466d0e1e0cc123afbe504da2462ca`
