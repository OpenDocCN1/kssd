
# Modem Spy v2.7

> 原文：[`www.pediy.com/kssd/pediy05/pediy50101.htm`](https://www.pediy.com/kssd/pediy05/pediy50101.htm)

软件名称：`Modem Spy v2.7`

软件介绍：`Modem Spy` 可以对网络电话进行谈话录音、纪录所有来电资料、软件内置自动应答功能、可以检测显示来电者的电脑 ID，录音的声音文件可存成 `MP3` 或 `WAV` 文件。（我都不知道如何用）

URL: `http://download.china.com/file/pc/nettool/netass/modemspyV2.7.exe`

手术刀：`SoftICE For Win2000`、`W32Dasm v10`(写破文要用它)、`fi`、`ProcDump`、`BCB 5.0`（写注册机）

手术时间：35 分 48 秒

难度：小鸡一只

这是我的第一篇破文，确实有点破，请多多包涵。我的第一个破解其实应该在一年以前了，可是后来一直没有深入，就被荒废了。前不久看了《软件加密及解密...》后，又有兴趣了。破解了好几个，心情一直不错。可上星期帮别人破解一个商业软件，用了一个星期都还没有完成（真不好意思，其实是自己水平太差）。想来就气，总要找几个出气筒。于是到“中华网-下载基地”去找出气筒，我下了三个 `<1M` 的（在大家都用 `ADSL` 的时候，我还在用 `56K` |:( ）共享软件。（人倒霉就连出气筒都不好找，我下了三个就有两个是免费的，好吧就从唯一一个开刀）。

先把程序安装好，用 `fi` 一测，嘿，居然用 `UPX` 加壳，这好办，用 `ProDump` 去壳。打开程序，进入注册窗口。输入用户名：`Shang Of Ghost` 序列号：`78787878`（嘿嘿，气吧，气吧...）。噢，差点忘了打开 `SoftICE`。用 `Bpx GetWindowTextA` 断点。敲两、三次看到下面的地址。

```
* Referenced by a CALL at Address:

|:004187DD 

|

:004187F0 56                         push esi
:004187F1 6880000000                 push 00000080
:004187F6 68F0F24200                 push 0042F2F0
:004187FB 8BF1                       mov esi, ecx
:004187FD 6A51                       push 00000051
:004187FF E88CCAFEFF                 call 00405290<===取得输入的用户名
:00418804 6880000000                 push 00000080
:00418809 6870F34200                 push 0042F370
:0041880E 6A53                       push 00000053
:00418810 8BCE                       mov ecx, esi
:00418812 E879CAFEFF                 call 00405290<===取得输入的注册码
:00418817 E824020000                 call 00418A40<===验证注册码
:0041881C 85C0                       test eax, eax
:0041881E 7434                       je 00418854<====这里跳走你就玩完了
:00418820 6870F34200                 push 0042F370
...
...
```

进入 `call 00418A40` 到这里：

```
...
...
:00418A6F 83C608                     add esi, 00000008
:00418A72 81FE28874200               cmp esi, 00428728
:00418A78 7CDC                       jl 00418A56
:00418A7A 6870F34200                 push 0042F370
:00418A7F 68F0F24200                 push 0042F2F0
:00418A84 E817FEFFFF                 call 004188A0<====这才是关键
:00418A89 83C408                     add esp, 00000008
:00418A8C 33C9                       xor ecx, ecx
:00418A8E 85C0                       test eax, eax
:00418A90 0F9FC1                     setg cl
:00418A93 5F                         pop edi
:00418A94 8BC1                       mov eax, ecx
:00418A96 5E                         pop esi
:00418A97 C3                         
```

进入 `Call 004188A0` 到这里：

```
* Referenced by a CALL at Address:

|:00418A84 

|

:004188A0 53                         push ebx
:004188A1 8B5C240C                   mov ebx, dword ptr [esp+0C]
:004188A5 56                         push esi
:004188A6 8B3570114200               mov esi, dword ptr [00421170]
:004188AC 57                         push edi
:004188AD 53                         push ebx
....
....这中间的没有什么用，好像是检查一些黑名单什么的
....
..
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00418919(C)

|

:004188E8 3C20                       cmp al, 20<===判断是否为空格，如果为空格就继续下一个字母
:004188EA 7427                       je 00418913
:004188EC 3C0D                       cmp al, 0D<===判断是否为回车，如果为空格就继续下一个字母
:004188EE 7423                       je 00418913
:004188F0 3C0A                       cmp al, 0A<===判断是否为结束，如果为空格就继续下一个字母
:004188F2 741F                       je 00418913
:004188F4 3C61                       cmp al, 61
:004188F6 7C0C                       jl 00418904
:004188F8 3C7A                       cmp al, 7A
:004188FA 7F08                       jg 00418904<===判断是否为小写字母，如果为小写字母，就转换为大写字母
:004188FC 0FBEC0                     movsx eax, al
:004188FF 83E820                     sub eax, 00000020
:00418902 EB03                       jmp 00418907

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004188F6(C), :004188FA(C)

|

:00418904 0FBEC0                     movsx eax, al

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00418902(U)

|

:00418907 8D1440                     lea edx, dword ptr [eax+2*eax]<==EAX 乘以 3
:0041890A C1E203                     shl edx, 03<===左移 3 位
:0041890D 2BD0                       sub edx, eax
:0041890F 8D741613                   lea esi, dword ptr [esi+edx+13]。<===将值保存在 esi 中

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004188EA(C), :004188EE(C), :004188F2(C)

|

:00418913 8A4101                     mov al, byte ptr [ecx+01]<===将下一个字母传到 al 中
:00418916 41                         inc ecx
:00418917 84C0                       test al, al<==判断字符串是否结束
:00418919 75CD                       jne 004188E8
:0041891B 85F6                       test esi, esi<===你要的注册码就在这里（注册码为十进制）
      在 SoftICE 中用? Esi 就可以看见
:0041891D 7D02                       jge 00418921
:0041891F F7DE                       neg esi
//以上是将输入的用户名转换为大写，再将每个字母的 ASCII 值乘以 3 再左移 3 位，    
减去本身的 ASCII，再加上 19，最后累加在 ESI 中

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004188E6(C), :0041891D(C)

|

:00418921 8A03                       mov al, byte ptr [ebx]
:00418923 8BCB                       mov ecx, ebx
:00418925 84C0                       test al, al
:00418927 7410                       je 00418939

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00418937(C)

|

:00418929 3C30                       cmp al, 30
:0041892B 7C04                       jl 00418931
:0041892D 3C39                       cmp al, 39
:0041892F 7E08                       jle 00418939

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041892B(C)

|

:00418931 8A4101                     mov al, byte ptr [ecx+01]
:00418934 41                         inc ecx
:00418935 84C0                       test al, al
:00418937 75F0                       jne 00418929

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00418927(C), :0041892F(C)

|

:00418939 51                         push ecx
:0041893A E833220000                 call 0041AB72<===将输入的注册码转换为数字值
      如："78787878"=78787878（十进制）
:0041893F 83C404                     add esp, 00000004
:00418942 33C9                       xor ecx, ecx
:00418944 3BC6                       cmp eax, esi<===比较输入的注册码的数字值是否和计算的值相等
      如果相等就注册成功（废话）
:00418946 0F94C1                     sete cl
:00418949 5F                         pop edi
:0041894A 5E                         pop esi
:0041894B 8BC1                       mov eax, ecx
:0041894D 5B                         pop ebx
:0041894E C3                         ret
```

大家懂了吗，唉我本来是为了解气，可是还是找到一个这么简单的，干脆把注册机一起写了吧。

```c
#include <vcl.h>

#pragma hdrstop

#include "Unit1.h"

//---------------------------------------------------------------------------
#pragma package(smart_init)

#pragma resource "*.dfm"

TForm1 *Form1;

//---------------------------------------------------------------------------
__fastcall TForm1::TForm1(TComponent* Owner)
        : TForm(Owner)
{
}

//---------------------------------------------------------------------------
void __fastcall TForm1::Button1Click(TObject *Sender)
{
unsigned i,j,m=0;
char *p;
if(Edit1->Text.Trim()=="")
{
        Application->MessageBox("姓名不能为空！","Modem Spy 注册码生成器",MB_OK);
        return ;
}
p=(Edit1->Text.UpperCase()).c_str();
for(i=0;i<strlen(p);i++)
{
        j=p[i];
        if(j!=0x20)
        {
                m=m+(((j*3)<<3)-j)+19;//就是这个 19 我一直把它输入为 13，害得我多呆了 15 分钟
        }
}
Edit2->Text=m;
}
//-------------------
```

还有没有看完的呀，（唉哟，别用石头砸我，....我还是遛了吧）。 `4dd466d0e1e0cc123afbe504da2462ca`
