
# MobileSearch(手机号码归属地查询) v2.0注册算法,附注册机~~~~~~

(30千字)

名称：MobileSearch(手机号码归属地查询) v2.0 测试版 数据库[2002-02-17]

下载：http://www.SearchCN.net

(作者主页)或天天安全网(正式版)

简介：本软件收录了34300多条手机号码归属地信息，可以详细查询到手机号码的开户地信息，包括该号码所属省份、城市、长途区号以及该号码SIM卡类型。号码资源收录容量极大，可以方便地帮你了解任何一个手机来电、短信息的来源地。

限制：在2002年2月3日前，您可以使用该注册测试版的全部功能，在2002年2月3日后，除了您不能使用在线更新数据库功能外，其他功能完全可以使用，但是，您无法获得我们的升级服务。

原因：不久前下载了MobileSearch v2.0 测试版,里面附有另类注册机,但不能用,于是自己动手,可能因为是测试版,它的保护很弱,输入注册码后会直接明码比较,所以找了个注册码自己使用.上个星期,在网上看到出了正式版,下载后发现保护加强,不直接比较,须退出后验证,试着把测试版的注册码填入可注册成功,说明它算法没变(已证实,请留意文章后部分),虽然可确定它的注册信息保存在`MobileSearch.ini`中,可下bpx `GetPrivateProfileString`来追,但感觉测试版比较容易入手,于是决定写个注册机以免日后麻烦,当追到它算法部分时更坚定了这个决心!

为何发布注册机：正式版中的使用条款上只说未注册不能使用超过一个月,不可修改本软件,不可使用修改版,不可获取和使用软件数据库,没说不能使用注册机:-)

另外我觉得,使用一个这么复杂烦琐而且应该不可逆的算法算出注册码,最后来个明码比较,这样的保护不如没有好,不过也难怪作者好象是第一次写软件,但愿以后我们的程序员都不要这样为好!同时说明一下如何用程序本身来写注册机

开始：软件是用Borland C++写的,没加壳,我的注册名Sam Von,我是用TRW直接bpm注册名找到下面代码的,用TRW而不用w32dasm的原因是可以比较容易确定那里才是算法的开始,因为程序可能会分几个部分来算注册码,缺点是很容易迷失方向.以下数据如不特别说明均为16进制数.另外要注意测试版未注册会在标题处有(注册版)字样,每次进入都会提示,注册成功后会消失!

## ==Main Code=================================================

这里比较明显,第1第2个Call是一样的,说明程序经过了两次相同的算法来获得正确的注册码,第3个Call是进行注册码比较的地方,很简单我没贴出来,我的目的是了解算法写出注册机

```
0187:004075D6  MOV      BYTE [EBP-77],00
0187:004075DA
PUSH    DWORD [EBP-70]    <---这里指向一个字串"MobileSearch2",重要数据
0187:004075DD  LEA      ECX,[EBP+FFFFFF6C] <---ecx指向我的注册名"Sam Von"
0187:004075E3  PUSH    ECX
0187:004075E4  CALL    00401C00
0187:004075E9  ADD      ESP,BYTE +08
0187:004075EC
MOV      [EBP-6C],EAX
0187:004075EF  PUSH
DWORD [EBP-70]    <---"MobileSearch2"
0187:004075F2  PUSH
  DWORD [EBP-6C]    <---这里指向上面Call算出的结果,称为
0187:004075F5
CALL    00401C00              中间注册码,用它代替注册名来运算
0187:004075FA  ADD      ESP,BYTE +08
0187:004075FD
MOV      [EBP-6C],EAX
0187:00407600  LEA
  EAX,[EBP+FFFFFF7C]
0187:00407606  PUSH    EAX
0187:00407607  PUSH    DWORD [EBP-6C]    <---这里就是正确的注册码了
0187:0040760A  CALL    004707F0
0187:0040760F  ADD
    ESP,BYTE +08
0187:00407612  TEST    EAX,EAX
0187:00407614  JNZ      NEAR 00407809
```

==Main Code
End================================================

## ==Main Call====================================================

这个算法可心说又臭又长,我写不出它的算法公式,光是在追Call 1的过程里我就做了4页纸的笔记,Call 2我就没做记录了,如果想了解它的算法请大家自己追一下,我只讲个大概:

1.程序先用字符"MobileSearch2"的前两位查表,得到两个数据,并将前两位当作注册码的前两位

2.将注册名的每一位X2,如Sam Von(53 61 6D 20 56 6F 6E)-->A6 C2 DA 40 AC DE DC 00

3.注册名最多只取8位,不够就用0补上

4.将2步得到的数据进行N次移位,补位,去位,前后4位相运算,查表得到"中间"注册码

5.重复1~4步,不过要将2中的注册名用"中间"注册码代替来运算,最后查表得正确注册码

注:下面注解中所说的"第2次运算"是指进入Main Code中(004075F5 CALL 00401C00)这个Call里内存的状态,因为虽然两个Call的过程一样,不过两次运算中内存的数据有了变化,调试过程中请注意[ebp+??]这样的地址!!

```
0187:00401C00  PUSH    EBP
0187:00401C01  MOV
    EBP,ESP
0187:00401C03  ADD      ESP,FFFFFF40
0187:00401C09  XOR      EAX,EAX
0187:00401C0B
MOV      [EBP-14],EAX
0187:00401C0E  XOR
  EDX,EDX
0187:00401C10  MOV      [EBP-18],EDX
        第1次运算这里为0,第二次运算这里是
0187:00401C13  LEA
    ECX,[EBP-38]      <---Call 2的运算结果37 8E 9C 0C
FD F9 75 96
0187:00401C16  MOV      [EBP-3C],ECX
        可先不管这一步
0187:00401C19  MOV
  EAX,[EBP+0C]
0187:00401C1C  CMP      BYTE [EAX],00
  <---[eax]="MobileSearch2"
0187:00401C1F  JNZ
  00401C25
0187:00401C21  MOV      DL,41
0187:00401C23
JMP      SHORT 00401C2A
0187:00401C25  MOV
  ECX,[EBP+0C]
0187:00401C28  MOV      DL,[ECX]
        <---"M"
0187:00401C2A  MOV
  [00487694],DL    <---这里是最后保存注册码的地方
0187:00401C30
XOR      EAX,EAX
0187:00401C32  MOV
AL,DL
0187:00401C34  MOV      [EBP-0C],EAX
0187:00401C37
MOV      ECX,[EBP-0C]
0187:00401C3A  XOR
  EAX,EAX
0187:00401C3C  MOV      AL,[ECX+0047E4B4]
<---用"M"查表,数据请参照注册机的数据DATA1
0187:00401C42  MOV
[EBP-14],EAX      <---Save
0187:00401C45  MOV
    EDX,[EBP+0C]
0187:00401C48  CMP      BYTE
[EDX+01],00
0187:00401C4C  JNZ      00401C52
0187:00401C4E
MOV      CL,41
0187:00401C50  JMP
SHORT 00401C58
0187:00401C52  MOV      EAX,[EBP+0C]
0187:00401C55  MOV      CL,[EAX+01]      <---"o"
0187:00401C58  MOV      [00487695],CL
0187:00401C5E
XOR      EAX,EAX
0187:00401C60  MOV
AL,CL
0187:00401C62  MOV      [EBP-0C],EAX
0187:00401C65
MOV      EDX,[EBP-0C]
0187:00401C68  XOR
  EAX,EAX
0187:00401C6A  MOV      AL,[EDX+0047E4B4]
<---用"o"查表,并shl它
0187:00401C70  SHL      EAX,04
0187:00401C73  MOV      [EBP-18],EAX      <---Save
0187:00401C76  XOR      EDX,EDX
0187:00401C78
MOV      [EBP-04],EDX
0187:00401C7B  MOV
  ECX,[EBP+08]
0187:00401C7E  MOV      AL,[ECX]
        <---"Sam Von"
0187:00401C80  MOV
    [EBP-3D],AL
0187:00401C83  INC      DWORD
[EBP+08]
0187:00401C86  CMP      BYTE [EBP-3D],00
0187:00401C8A  JZ      00401CA1
0187:00401C8C  MOV
    DL,[EBP-3D]
0187:00401C8F  ADD      EDX,EDX
        <---ADD
0187:00401C91  MOV
  ECX,[EBP-04]
0187:00401C94  MOV      [EBP+ECX-2C],DL
0187:00401C98  INC      DWORD [EBP-04]
0187:00401C9B
CMP      DWORD [EBP-04],BYTE +08
0187:00401C9F  JC
    00401C7B
0187:00401CA1  CMP      DWORD
[EBP-04],BYTE +08
0187:00401CA5  JNC      00401CB8
0187:00401CA7  MOV      EAX,[EBP-04]
0187:00401CAA
MOV      BYTE [EBP+EAX-2C],00
0187:00401CAF  INC
    DWORD [EBP-04]    <---最多取8位,不够补0
0187:00401CB2
CMP      DWORD [EBP-04],BYTE +08
0187:00401CB6  JC
    00401CA7
0187:00401CB8  LEA      EDX,[EBP+FFFFFF40]
0187:00401CBE  PUSH    EDX
0187:00401CBF  LEA
    ECX,[EBP-2C]
0187:00401CC2  PUSH    ECX
0187:00401CC3  CALL    00401514
<---Call 1先进去看看
0187:00401CC8  ADD      ESP,BYTE +08
0187:00401CCB  PUSH    DWORD [EBP-18]<---340
0187:00401CCE
PUSH    DWORD [EBP-14]<---18
0187:00401CD1  LEA
  EAX,[EBP+FFFFFF40]  <---指向Call 1结果 lea eax,[esp]=mov eax,esp
0187:00401CD7  PUSH    EAX
0187:00401CD8  LEA
  EDX,[EBP-1C]
0187:00401CDB  PUSH    EDX
0187:00401CDC
LEA      ECX,[EBP-20]
0187:00401CDF  PUSH
ECX
0187:00401CE0  CALL    00401825      <---进入Call
2
0187:00401CE5  ADD      ESP,BYTE +14  <---esp指向Call
2结果
0187:00401CE8  MOV      EAX,[EBP-20]  <---Call
2结果前4位
0187:00401CEB  MOV      [EBP-24],EAX
0187:00401CEE
MOV      DL,[EBP-24]
0187:00401CF1  AND
  DL,FF
0187:00401CF4  MOV      ECX,[EBP-3C]
0187:00401CF7  MOV      [ECX],DL
0187:00401CF9
INC      DWORD [EBP-3C]
0187:00401CFC  MOV
  EAX,[EBP-24]
0187:00401CFF  SHR      EAX,08
0187:00401D02  AND      AL,FF
0187:00401D04  MOV
    EDX,[EBP-3C]
0187:00401D07  MOV      [EDX],AL
0187:00401D09  INC      DWORD [EBP-3C]
0187:00401D0C
MOV      ECX,[EBP-24]
0187:00401D0F  SHR
  ECX,10
0187:00401D12  AND      CL,FF
0187:00401D15
MOV      EAX,[EBP-3C]
0187:00401D18  MOV
  [EAX],CL
0187:00401D1A  INC      DWORD [EBP-3C]
0187:00401D1D  MOV      EDX,[EBP-24]
0187:00401D20
SHR      EDX,18
0187:00401D23  AND
DL,FF
0187:00401D26  MOV      ECX,[EBP-3C]
0187:00401D29
MOV      [ECX],DL
0187:00401D2B  INC
DWORD [EBP-3C]
0187:00401D2E  MOV      EAX,[EBP-1C]
0187:00401D31  MOV      [EBP-24],EAX
0187:00401D34
MOV      DL,[EBP-24]
0187:00401D37  AND
  DL,FF
0187:00401D3A  MOV      ECX,[EBP-3C]
0187:00401D3D  MOV      [ECX],DL
0187:00401D3F
INC      DWORD [EBP-3C]
0187:00401D42  MOV
  EAX,[EBP-24]
0187:00401D45  SHR      EAX,08
0187:00401D48  AND      AL,FF
0187:00401D4A  MOV
    EDX,[EBP-3C]
0187:00401D4D  MOV      [EDX],AL
0187:00401D4F  INC      DWORD [EBP-3C]
0187:00401D52
MOV      ECX,[EBP-24]
0187:00401D55  SHR
  ECX,10
0187:00401D58  AND      CL,FF
0187:00401D5B
MOV      EAX,[EBP-3C]
0187:00401D5E  MOV
  [EAX],CL
0187:00401D60  INC      DWORD [EBP-3C]
0187:00401D63  MOV      EDX,[EBP-24]
0187:00401D66
SHR      EDX,18
0187:00401D69  AND
DL,FF
0187:00401D6C  MOV      ECX,[EBP-3C]
0187:00401D6F
MOV      [ECX],DL
0187:00401D71  INC
DWORD [EBP-3C]
0187:00401D74  XOR      EAX,EAX
0187:00401D76
MOV      [EBP-10],EAX
0187:00401D79  MOV
  BYTE [EBP-3E],80  <---这几个数据是要参与运算的
0187:00401D7D  MOV
    BYTE [EBP-30],00
0187:00401D81  MOV
DWORD [EBP-04],02
```
