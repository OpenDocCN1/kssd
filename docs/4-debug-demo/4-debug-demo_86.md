

# crackcode 突破 25 位限制分析

> 原文：[`www.pediy.com/kssd/pediy03/forum1263.htm`](https://www.pediy.com/kssd/pediy03/forum1263.htm)

各位大虾好：   
    我是当当虾，对了，那外号为"豆豆虾"的是我非常好的网友，最近他从事销售   
要出远门，跑客户，所以不会常来，我暂时接替他。他一句话：都是为了生活。   
生活本来就是很乱，的确啊！还是古人云的好：生活本来就是一团麻绳，理也理不清。   
呵呵～～～但是有一点是使我本人都很欣慰的事情：很开心，而且有空的时间可以做   
自己喜欢的事，一句话"走自己的路，让别人羡慕去吧！"（据说，这句话是出自一位   
crack 老前辈对新生辈出年代的询询教诲！ ）   
  
  
        话说回来，想当初我的好友"豆豆虾"，一来看雪论坛就"炸"道，我想也许是有   
两点原因在这里解释一下：   
1、无论如何，人都是有虚荣心，想出点小名很正常，其实一个主要原因想得到别人的认   
可。我本人比较欣赏通过自己的实力去得到别人的认可。   
2、"炸"道本身谐音，有点标新立异，其根本的就是在众目睽睽下，想吸引别人的注意。   
芸芸众生过往云烟。所以啊！比如这一招式，在追女仔中非常常见，好好体会和掌握吧！   
哈哈～～～ 我和"豆豆虾"都是光有理论无实践。 估计有人要吐啦。。。。哈哈～～   
          
        我当当虾在这表示歉意。此次的目的，是想把"炸坏的道路给修一修"，   
"将功补过"吧！豆豆虾曾经把修改 crackcode 的过程讲述过，经过我的再跟踪，写写凑凑   
一番，"文物"得以出土问世。哈哈～～～～   
  
【如何突破 crackcode 的 25 位限制】   
1、突破 crackcode 的 25 位限制的经过   
    在论坛中，有一些贴子常提到 crackcode 存在 25 位的限制。可是 rufeng 又隐退江湖，   
（江湖盛传他可能在修炼内功，准备东山再起）。于是好友豆豆虾（学过一点汇编，知道   
一些常用工具的使用），凡心欲动，本意是想一一步分析，对代码原理有个总的了解。   
然后再对证下药。所谓手到病除。他说如果一一分析下去，也许在作修改的时候，就不   
会在一个数据结构上兜圈子，当时猜想 rufeng 是定义一个 25 位的字符数组，没有考虑到   
获得的结果会超过 25 位的情况。 【爆破，可以采取直捣黄龙；作注册机，得看懂整个注册   
码；修改代码，得看懂程序的实现原理，然后再 OEM 自己的思路。】       
  
2、为什么 crackcode 存在 25 位限制（打算介绍简单些，因为它牵扯到的面比较广）   
        crackcode 调用 getthreadcontext 函数，返回一个结构指针，利用其读取被跟踪   
程序的寄存器（eax、edx...)中的值（这个函数的功能非常的棒）。rufeng 巧妙的应用   
了六个寄存器（Edi，Esi，Ebx，Edx，Ecx，Eax），返回它们注册码，一个寄存器   
只能存 4 个字符，所以 6\*4=24，确切的说，crackcode 取到的字符长度不能超过 24 个。   
比如 EAX=44324432--->储存"D2D2"四个字符。   
  
3、如何突破它的限制？？   
    如果把程序通读一遍，似乎发现没有办法去突破这个限制，因为这是设计上的   
失误。如果细细去琢磨 crackcode，会发现它两次调用 getthreadcontext 函数。其实第   
一次的时候，就可以得到要读取注册码的地址。根据 crackcode.ini 中 Save\_Code\_Address   
的值，对应寄存器储存的值。为什么 crackcode 要去两次调用 getthreadcontext 函数，其中   
的奥秘，把程序认认真真的通读几遍（读到凌晨 4 点，窗外一片寂寂。在路灯下，几个清洁   
工的身影，默默无闻的清洗大地，在我们匆匆来往的街道，还以清爽的感觉。）   
好了！上面既然第一次调用的时候，已经能确定注册码的位置，何不拿它示问。于是可以   
调用 ReadProcessMemory，把注册码得到。   
  
4、修改程序（代码修改位置 401448－－－40149B）   
    我这有豆豆虾修改时的草稿：   
＝＝＝＝＝＝＝＝＝判断 save\_code\_address 需要读取的寄存器＝＝＝＝＝＝＝＝＝＝＝       
\----deal with ebp\----   
mov ebx,40a64c  －－－－－－－crackcode 对 save\_code\_address 处理后存放的地址   
mov ecx,40a1cc   
mov al,byte ptr \[ebx\]   
cmp al,0f5      －－－－－－－为什么要进行这样比较，crackcode 代码分享笔记(三)中   
            有讲述。我把相关的部分放在附录中。   
je final   
\----deal with eax\----   
sub ecx,4   
cmp al,0f0   
je final   
\----deal with ecx\----   
sub ecx,4   
cmp al,0f1   
je final   
\----deal with edx\----   
sub ecx,4   
cmp al,0f2   
je final   
\----deal with ebx\----   
sub ecx,4   
cmp al,0f3   
je final   
\----deal with esi\----   
sub ecx,4   
cmp al,090   
je final   
\----deal with edi\----   
sub ecx,4   
//cmp al,0f7   
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝   
＝＝＝＝＝＝＝＝＝＝以下为从内存中读取注册码，并显示＝＝＝＝＝＝＝＝   
final：   
  
\---deal with mem---   
push 0   
push 28 //size   
push 409B80  //buffer    ======缓冲区   
push d,\[ecx\] //baseaddr  ======存放注册码的地址。   
push d,\[40a410\] //hprocess   
call d,\[405024\]      ====调用 ReadProcessMemory.从存放注册码的地址读入缓冲区   
\---deal with mem---   
  
display:   
  mov ebx,409B80  ＝＝＝显示缓冲区   
  jmp 40149c   
  
结束语：所有的人都恐惧孤独，但唯有在孤独之中，我们才能学会了解自己，学习如何处理   
我们自己永恒的孤独。 ：）   
【爆破，可以采取直捣黄龙；作注册机，得看懂整个注册码；   
修改代码，得看懂程序的实现原理，然后再 OEM 自己的思路。】   
          
                                                                                    当当虾      耗时：6 小时   
                            2001 年 8 月 24 日 18:07    
附录：   
  
\* Possible StringData Ref from Data Obj ->"EAX"   
                                  |   
:004011A7 68AC614000              push 004061AC   
:004011AC 56                      push esi      －－》"Save\_Code\_Address"   
:004011AD E81E080000              call 004019D0   
:004011B2 59                      pop ecx   
:004011B3 85C0                    test eax, eax   
:004011B5 59                      pop ecx   
:004011B6 7511                    jne 004011C9   
:004011B8 C605609740008B          mov byte ptr \[00409760\], 8B   
:004011BF C7054CA64000F0909090    mov dword ptr \[0040A64C\], 909090F0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004011B6(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"EBX"   
                                  |   
:004011C9 68A8614000              push 004061A8   
:004011CE 56                      push esi   
:004011CF E8FC070000              call 004019D0   
:004011D4 59                      pop ecx   
:004011D5 85C0                    test eax, eax   
:004011D7 59                      pop ecx   
:004011D8 7511                    jne 004011EB   
:004011DA C605609740008B          mov byte ptr \[00409760\], 8B   
:004011E1 C7054CA64000F3909090    mov dword ptr \[0040A64C\], 909090F3   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004011D8(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"ECX"   
                                  |   
:004011EB 68A4614000              push 004061A4   
:004011F0 56                      push esi   
:004011F1 E8DA070000              call 004019D0   
:004011F6 59                      pop ecx   
:004011F7 85C0                    test eax, eax   
:004011F9 59                      pop ecx   
:004011FA 7511                    jne 0040120D   
:004011FC C605609740008B          mov byte ptr \[00409760\], 8B   
:00401203 C7054CA64000F1909090    mov dword ptr \[0040A64C\], 909090F1   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004011FA(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"EDX"   
                                  |   
:0040120D 68A0614000              push 004061A0   
:00401212 56                      push esi   
:00401213 E8B8070000              call 004019D0   
:00401218 59                      pop ecx   
:00401219 85C0                    test eax, eax   
:0040121B 59                      pop ecx   
:0040121C 7511                    jne 0040122F   
:0040121E C605609740008B          mov byte ptr \[00409760\], 8B   
:00401225 C7054CA64000F2909090    mov dword ptr \[0040A64C\], 909090F2   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040121C(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"EDI"   
                                  |   
:0040122F 689C614000              push 0040619C   
:00401234 56                      push esi   
:00401235 E896070000              call 004019D0   
:0040123A 59                      pop ecx   
:0040123B 85C0                    test eax, eax   
:0040123D 59                      pop ecx   
:0040123E 7511                    jne 00401251   
:00401240 C605609740008B          mov byte ptr \[00409760\], 8B   
:00401247 C7054CA64000F7909090    mov dword ptr \[0040A64C\], 909090F7   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040123E(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"ESI"   
                                  |   
:00401251 6898614000              push 00406198   
:00401256 56                      push esi   
:00401257 E874070000              call 004019D0   
:0040125C 59                      pop ecx   
:0040125D 85C0                    test eax, eax   
:0040125F 59                      pop ecx   
:00401260 7511                    jne 00401273   
:00401262 C6056097400090          mov byte ptr \[00409760\], 90   
:00401269 C7054CA6400090909090    mov dword ptr \[0040A64C\], 90909090   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401260(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"EBP"   
                                  |   
:00401273 6894614000              push 00406194   
:00401278 56                      push esi   
:00401279 E852070000              call 004019D0   
:0040127E 59                      pop ecx   
:0040127F 85C0                    test eax, eax   
:00401281 59                      pop ecx   
:00401282 7511                    jne 00401295   
:00401284 C605609740008B          mov byte ptr \[00409760\], 8B   
:0040128B C7054CA64000F5909090    mov dword ptr \[0040A64C\], 909090F5   
＝＝＝＝以上程序是对处理 Save\_Code\_Address 不同值，做相应的处理＝＝＝＝＝＝   
相当于   
switch( Save\_Code\_Address )   
{   
    case 'EAX':   
        code1=8B;   
    code2=909090F0；   
    break;   
    case 'EBX':   
        code1=8B;   
    code2=909090F0；   
    break;   
    case 'EC':   
        code1=8B;   
    code2=909090F0；   
    break;   
    case 'EDX':   
        code1=8B;   
    code2=909090F0；   
    break;   
    case 'EDI':   
        code1=8B;   
    code2=909090F0；   
    break;   
    case 'ESI':   
        code1=90;   
    code2=909090F0；   
    break;   
    case 'EBP':   
        code1=8B;   
    code2=909090F0；   
    break;   
  
} 

各位大虾好：   
    我是当当虾，上回贴子中已经介绍了“如何突破 crackcode 的 25 位限制”，接下来   
，继续讲述在这基础上豆豆虾如何修改 crackcode，把它变成具有粘贴功能！   
  
【如何让 crackcode 变得具有粘贴功能】   
前言：   
    整个过程当成一个 project（项目），真的可以过把瘾哦！：）   
  
1、如何让 crackcode 变得具有粘贴功能的经过   
              当初豆豆虾修改成突破 25 位后，无意中看到一个贴，说到 crackcode 不具有粘贴   
功能，使用起来特不方便的。豆豆虾想，嘿！还真的是挺实用的一个功能。开始琢磨如何   
修改代码。这其中的过程，从琢磨、修改、调试自至成功。对于豆豆虾来说，充满乐趣   
和熬夜。但我想，如果能把乐趣，与更多人来分享，那不是更快乐吗？！人生几何，自   
己觉好的东东，暂且不论其好与坏，都可以拿出来与别人分享，也许希望能马上得到认   
可，如果遭到冷嘲热讽后，而不继续前行，会觉很可惜。还是那一句老话－－"走自己的   
路，让别人说去吧！"时间可以证明一切（ 呵呵～～～，两篇文章始终围绕这个中心思想   
哦！如果能我有机会再碰到，从前教过我语文的老师，或许可以拿到个 60 分及格哦！哈   
哈～～。我非常留念从前的老师、朋友、同学，如果有机会在一起的，多好，如今都各   
奔东西。前几天电视采访大运大使杨澜时，她说常在梦中梦见最多的是，自己的校   
园，每个角落，和同学们度过美好时光。珍惜在你自己周围的朋友、亲人吧！一份汗水、   
一份喜悦；一份帮助，一份友情。   
  
2、编写具有粘贴功能的程序   
              需要加入粘贴功能，必须得知道如何编写具有粘贴功能的程序，以及实现的原   
理。   
              豆豆虾说“简单，在 internet 上，随时随地都能找到你需要的东西”，于是找到了如   
下的程序：   
how to place text on the clipboard   
  
    cstring source;   
    //put your text in source   
    if(openclipboard())   
    {   
        hglobal clipbuffer;   
        char \* buffer;   
        emptyclipboard();   
        clipbuffer = globalalloc(gmem\_ddeshare, source.getlength()+1);   
        buffer = (char\*)globallock(clipbuffer);   
        strcpy(buffer, lpcstr(source));   
        globalunlock(clipbuffer);   
        setclipboarddata(cf\_text,clipbuffer);   
        closeclipboard();   
    }   
how to get text off of the clipboard   
this is easy really but here it is for completeness   
  
  
    char \* buffer;   
    if(openclipboard())   
    {   
           
        buffer = (char\*)getclipboarddata(cf\_text);   
        //do something with buffer here   
        //before it goes out of scope   
           
    }   
  
    closeclipboard();   
  
  
  
对以上的代码作了小小的调整。如何使用这段代码？在 vc 中自动生成一个基于于对话框   
程序，双击 ok 按钮，把以下程序粘贴过去，这样大功告成。运行程序，点击 ok 键，这样   
在剪贴板中的内容就是“test for clipbroad!!”   
void CClipbroadDlg::OnOK()   
{   
    // TODO: Add extra validation here   
    CString source("test for clipbroad!!!");    //初始化 source    
    //put your text in source   
    if(OpenClipboard())                              //打开剪贴板   
    {   
        HGLOBAL clipbuffer;   
        char \* buffer;   
        clipbuffer = GlobalAlloc(GMEM\_DDESHARE, source.GetLength()+1);   
                                                      //分配一段 dde 内存   
            EmptyClipboard();    //清空剪贴板   
        buffer = (char\*)GlobalLock(clipbuffer);  //锁定申请到的内存   
        strcpy(buffer, LPCSTR(source));          //复制字符串到内存   
        GlobalUnlock(clipbuffer);                      //解锁内存     
        SetClipboardData(CF\_TEXT,clipbuffer); //设定剪贴板的内容   
        CloseClipboard(); ｝                          //关闭剪贴板    
}   
  
3、转换 vc 程序为汇编程序。   
    为什么要这样呢？？原因有两点：一、对于 CF\_TEXT 之类的宏，不知道到底是代表什么   
值   
                        二、汇编程序反编译可以非常简单的照搬到程序中。   
    我把 iczelion 的第一个 masm 的 ok 演示程序修改成如下：   
  .386   
.model flat, stdcall   
option casemap:none   
include \\masm32\\include\\windows.inc   
include \\masm32\\include\\kernel32.inc   
include \\masm32\\include\\user32.inc   
includelib \\masm32\\lib\\user32.lib   
includelib \\masm32\\lib\\kernel32.lib   
  
.data   
  
MsgCaption      db "Iczelion's tutorial no.2",0   
MsgBoxText      db "Win32 Assembly is Great!",0   
  
  
.data?   
hMemory HANDLE ?   
pMemory DWORD ?   
  
  
.code   
start:   
       
invoke OpenClipboard,NULL  ;打开剪贴板   
test eax,eax   
je endpro   
mov edi,offset MsgCaption  ; 取得标题，把它当成剪贴板中的内容   
or ecx,0FFFFFFFFh   
xor eax,eax   
repnz scasb                  
not ecx                  ;得到标题长度   
sub edi,ecx   
push ecx   
push edi              ;这两个 push 也许不需要。没有时间去尝试   
inc ecx   
invoke GlobalAlloc,GMEM\_DDESHARE,ecx  ;锁定申请到的内存   
mov hMemory,eax   
invoke EmptyClipboard                ;/清空剪贴板   
invoke GlobalLock,hMemory      ;锁定申请到的内存   
mov  pMemory,eax   
pop esi   
pop ecx   
mov edi,eax   
rep movs byte ptr \[edi\],byte ptr \[esi\]  ;/复制字符串到内存   
  
invoke GlobalUnlock,pMemory          ;解锁内存   
invoke SetClipboardData,CF\_TEXT,pMemory;  ;设定剪贴板的内容   
invoke CloseClipboard                                      ;关闭剪贴板   
endpro:   
    invoke MessageBox, NULL,addr MsgBoxText, addr MsgCaption, MB\_OK   
    invoke ExitProcess,NULL   
end start   
  
4、再看看反编译的结果：（其中一个目的是想得到需要宏的数值，其实也有简单   
的方法，就是查询 windows.h）   
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Program Entry Point \*\*\*\*\*\*\*\*   
:00401000 6A00                    push 00000000   
  
\* Reference To: USER32.OpenClipboard, Ord:01D0h   
                                  |   
:00401002 E88B000000              Call 00401092   
:00401007 85C0                    test eax, eax   
:00401009 745B                    je 00401066   
  
\* Possible StringData Ref from Data Obj ->"Iczelion's tutorial no.2"   
                                  |   
:0040100B BF00304000              mov edi, 00403000   
:00401010 83C9FF                  or ecx, FFFFFFFF   
:00401013 33C0                    xor eax, eax   
:00401015 F2                      repnz   
:00401016 AE                      scasb   
:00401017 F7D1                    not ecx   
:00401019 2BF9                    sub edi, ecx   
:0040101B 51                      push ecx   
:0040101C 57                      push edi   
:0040101D 41                      inc ecx   
:0040101E 51                      push ecx   
:0040101F 6800200000              push 00002000 //,GMEM\_DDESHARE   
  
\* Reference To: KERNEL32.GlobalAlloc, Ord:0168h   
                                  |   
:00401024 E87B000000              Call 004010A4   
:00401029 A334304000              mov dword ptr \[00403034\], eax   
  
\* Reference To: USER32.EmptyClipboard, Ord:00B3h   
                                  |   
:0040102E E853000000              Call 00401086   
:00401033 FF3534304000            push dword ptr \[00403034\]   
  
\* Reference To: KERNEL32.GlobalLock, Ord:0173h   
                                  |   
:00401039 E86C000000              Call 004010AA   
:0040103E A338304000              mov dword ptr \[00403038\], eax   
:00401043 5E                      pop esi   
:00401044 59                      pop ecx   
:00401045 8BF8                    mov edi, eax   
:00401047 F3                      repz   
:00401048 A4                      movsb   
:00401049 FF3538304000            push dword ptr \[00403038\]   
  
\* Reference To: KERNEL32.GlobalUnlock, Ord:017Ah   
                                  |   
:0040104F E85C000000              Call 004010B0   
:00401054 FF3538304000            push dword ptr \[00403038\]   
:0040105A 6A01                    push 00000001  //,CF\_TEXT   
  
\* Reference To: USER32.SetClipboardData, Ord:021Fh   
                                  |   
:0040105C E837000000              Call 00401098   
  
\* Reference To: USER32.CloseClipboard, Ord:003Bh   
                                  |   
:00401061 E81A000000              Call 00401080   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401009(C)   
|   
:00401066 6A00                    push 00000000   
  
\* Possible StringData Ref from Data Obj ->"Iczelion's tutorial no.2"   
                                  |   
:00401068 6800304000              push 00403000   
  
\* Possible StringData Ref from Data Obj ->"Win32 Assembly is Great!"   
                                  |   
:0040106D 6819304000              push 00403019   
:00401072 6A00                    push 00000000   
  
\* Reference To: USER32.MessageBoxA, Ord:01BBh   
                                  |   
:00401074 E813000000              Call 0040108C   
:00401079 6A00                    push 00000000   
  
\* Reference To: KERNEL32.ExitProcess, Ord:0075h   
                                  |   
:0040107B E81E000000              Call 0040109E   
  
5、寻找空间插入代码   
    前面做了如此多的准备工作，都是为了这一步。正所谓，现实中机会是   
很多，关键你的知识是否已经准备充足呢？？：）   
                首先使用 topo 软件，进行测量 crackcode 中是否有空间插入代码。然后再使用   
hiew 进行静态的写入代码。运行前，需要把代码的 SECTION 属性，通过 pebulid 编辑   
成可读可写的。   
                通过 topo 得到空间后，接下来你需要做一个小小的计划。   
1、404400 －－－插入代码的开始   
2、404900 －－－调用函数的地址   
3、404A00－－－调用的函数名   
                  同时，调用的函数入口，必须得通过 getprocessaddress，getmodulehandled   
得到。   
以下是豆豆虾的笔记：   
  
getprocessaddress  call dword ptr \[4050A8\]   
getmodulehandled call dword ptr \[405038\]   
KERNEL32.DLL-------405530   
USER32.DLL-----------40553d   
plan :   
1、404400－－－code   
2、404900－－－address of function   
3、404A00－－－function's name   
  
404A00－－－function's name   
  
▓ 00004A00:  4F 70 65 6E-43 6C 69 70-62 6F 61 72-64 00 90 90  OpenClipboard éé   
▓ 00004A10:  45 6D 70 74-79 43 6C 69-70 62 6F 61-72 64 00 90  EmptyClipboard é   
▓ 00004A20:  47 6C 6F 62-61 6C 41 6C-6C 6F 63 00-90 90 90 90  GlobalAlloc éééé   
▓ 00004A30:  47 6C 6F 62-61 6C 4C 6F-63 6B 00 90-90 90 90 90  GlobalLock ééééé   
▓ 00004A40:  47 6C 6F 62-61 6C 55 6E-6C 6F 63 6B-00 90 90 90  GlobalUnlock ééé   
▓ 00004A50:  53 65 74 43-6C 69 70 62-6F 61 72 64-44 61 74 61  SetClipboardData   
▓ 00004A60:  00 90 90 90-90 90 90 90-90 90 90 90-90 90 90 90  ééééééééééééééé   
▓ 00004A70:  43 6C 6F 73-65 43 6C 69-70 62 6F 61-72 64 00 90  CloseClipboard é   
  
404900－－－address of function   
OpenClipboard    =  404900  ---user32.dll   
EmptyClipboard    =  404904  ---user32.dll   
GlobalAlloc          =  404908      ---kernel32.dll   
GlobalLock    =  40490C      ---kernel32.dll   
GlobalUnlock    =  404910      ---kernel32.dll   
SetClipboardData    =  404914  ---user32.dll   
CloseClipboard    =  404918  ---user32.dll                
  
hmemory         =  40491C    var   
pmemory        =  404920  var   
  
    好了，有了如此之多的准备，该开始修改了，八年抗战该结束了   
呵呵～～～   
＝＝＝＝＝＝＝＝＝＝突破 24 位的修改＝＝＝＝＝＝＝＝＝＝＝＝＝   
:00401448 BB4CA64000              mov ebx, 0040A64C   
:0040144D B9CCA14000              mov ecx, 0040A1CC   
:00401452 8A03                    mov al, byte ptr \[ebx\]   
:00401454 3CF5                    cmp al, F5   
:00401456 7426                    je 0040147E   
:00401458 83E904                  sub ecx, 00000004   
:0040145B 3CF0                    cmp al, F0   
:0040145D 741F                    je 0040147E   
:0040145F 83E904                  sub ecx, 00000004   
:00401462 3CF1                    cmp al, F1   
:00401464 7418                    je 0040147E   
:00401466 83E904                  sub ecx, 00000004   
:00401469 3CF2                    cmp al, F2   
:0040146B 7411                    je 0040147E   
:0040146D 83E904                  sub ecx, 00000004   
:00401470 3CF3                    cmp al, F3   
:00401472 740A                    je 0040147E   
:00401474 83E904                  sub ecx, 00000004   
:00401477 3C90                    cmp al, 90   
:00401479 7403                    je 0040147E   
:0040147B 83E904                  sub ecx, 00000004   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00401456(C), :0040145D(C), :00401464(C), :0040146B(C), :00401472(C)   
|:00401479(C)   
|   
:0040147E 6A00                    push 00000000   
:00401480 6A28                    push 00000028   
:00401482 68809B4000              push 00409B80   
:00401487 FF31                    push dword ptr \[ecx\]   
:00401489 FF3510A44000            push dword ptr \[0040A410\]   
  
\* Reference To: KERNEL32.ReadProcessMemory, Ord:0000h   
                                  |   
:0040148F FF1524504000            Call dword ptr \[00405024\]   
:00401495 BB809B4000              mov ebx, 00409B80   
:0040149A E9612F0000              jmp 00404400   
＋＋＋＋＋＋＋＋＋这句 jmp 到插入的代码处＋＋＋＋＋＋＋＋＋＋＋＋＋＋＋＋   
:0040149F 90                      nop   
:004014A0 90                      nop   
:004014A1 90                      nop   
:004014A2 90                      nop   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404504(U)   
|   
:004014A3 53                      push ebx   
:004014A4 6A00                    push 00000000   
  
\* Reference To: USER32.MessageBoxA, Ord:0000h   
                                  |   
:004014A6 FF15C0504000            Call dword ptr \[004050C0\]   
:004014AC E8BB040000              call 0040196C   
＝＝＝＝＝＝＝＝＝突破 24 位的修改＝＝＝＝＝＝＝＝＝＝＝＝＝＝                  
  
＝＝＝＝＝＝＝＝＝增加粘贴功能＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝   
:00404400 6660                    pusha   
:00404402 669C                    pushf          //保护寄存器   
－－－－－－－－－取得各个函数的入口－－－－－－－－－－－－   
:00404404 683D554000              push 0040553D   
  
\* Reference To: KERNEL32.GetModuleHandleA, Ord:0000h   
                                  |   
:00404409 FF1538504000            Call dword ptr \[00405038\]   
:0040440F 8BF8                    mov edi, eax   
  
\* Possible StringData Ref from Code Obj ->"OpenClipboard"   
                                  |   
:00404411 68004A4000              push 00404A00   
:00404416 57                      push edi   
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                                  |   
:00404417 FF15A8504000            Call dword ptr \[004050A8\]   
:0040441D A300494000              mov dword ptr \[00404900\], eax   
  
\* Possible StringData Ref from Code Obj ->"EmptyClipboard"   
                                  |   
:00404422 68104A4000              push 00404A10   
:00404427 57                      push edi   
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                                  |   
:00404428 FF15A8504000            Call dword ptr \[004050A8\]   
:0040442E A304494000              mov dword ptr \[00404904\], eax   
  
\* Possible StringData Ref from Code Obj ->"SetClipboardData"   
                                  |   
:00404433 68504A4000              push 00404A50   
:00404438 57                      push edi   
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                                  |   
:00404439 FF15A8504000            Call dword ptr \[004050A8\]   
:0040443F A318494000              mov dword ptr \[00404918\], eax   
  
\* Possible StringData Ref from Code Obj ->"CloseClipboard"   
                                  |   
:00404444 68704A4000              push 00404A70   
:00404449 57                      push edi   
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                                  |   
:0040444A FF15A8504000            Call dword ptr \[004050A8\]   
:00404450 A314494000              mov dword ptr \[00404914\], eax   
:00404455 6830554000              push 00405530   
  
\* Reference To: KERNEL32.GetModuleHandleA, Ord:0000h   
                                  |   
:0040445A FF1538504000            Call dword ptr \[00405038\]   
:00404460 8BF8                    mov edi, eax   
  
\* Possible StringData Ref from Code Obj ->"GlobalAlloc"   
                                  |   
:00404462 68204A4000              push 00404A20   
:00404467 57                      push edi   
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                                  |   
:00404468 FF15A8504000            Call dword ptr \[004050A8\]   
:0040446E A308494000              mov dword ptr \[00404908\], eax   
  
\* Possible StringData Ref from Code Obj ->"GlobalLock"   
                                  |   
:00404473 68304A4000              push 00404A30   
:00404478 57                      push edi   
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                                  |   
:00404479 FF15A8504000            Call dword ptr \[004050A8\]   
:0040447F A30C494000              mov dword ptr \[0040490C\], eax   

> \* Possible StringData Ref from Code Obj ->"GlobalUnlock"   
>                                   |   
> :00404484 68404A4000              push 00404A40   
> :00404489 57                      push edi   
>   
> \* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
>                                   |   
> :0040448A FF15A8504000            Call dword ptr \[004050A8\]   
> :00404490 A310494000              mov dword ptr \[00404910\], eax   
> －－－以下为剪贴板调用（注意在程序中没有对错误的结果进行判断）－－－－－－   
> :00404495 6A00                    push 00000000   
> :00404497 FF1500494000            call dword ptr \[00404900\]   
> :0040449D FF1504494000            call dword ptr \[00404904\]   
> :004044A3 BF809B4000              mov edi, 00409B80   
> :004044A8 83C9FF                  or ecx, FFFFFFFF   
> :004044AB 33C0                    xor eax, eax   
> :004044AD F2                      repnz   
> :004044AE AE                      scasb   
> :004044AF F7D1                    not ecx   
> :004044B1 2BF9                    sub edi, ecx   
> :004044B3 57                      push edi   
> :004044B4 51                      push ecx   
> :004044B5 41                      inc ecx   
> :004044B6 51                      push ecx   
> :004044B7 6800200000              push 00002000   
> :004044BC FF1508494000            call dword ptr \[00404908\]   
> :004044C2 A31C494000              mov dword ptr \[0040491C\], eax   
> :004044C7 50                      push eax   
> :004044C8 FF150C494000            call dword ptr \[0040490C\]   
> :004044CE A320494000              mov dword ptr \[00404920\], eax   
> :004044D3 59                      pop ecx   
> :004044D4 5E                      pop esi   
> :004044D5 8BF8                    mov edi, eax   
> :004044D7 F3                      repz   
> :004044D8 A4                      movsb   
> :004044D9 FF3520494000            push dword ptr \[00404920\]   
> :004044DF FF1510494000            call dword ptr \[00404910\]   
> :004044E5 FF3520494000            push dword ptr \[00404920\]   
> :004044EB 6A01                    push 00000001   
> :004044ED FF1518494000            call dword ptr \[00404918\]   
> :004044F3 FF1514494000            call dword ptr \[00404914\]   
> －－－－－－－－－－－以下为恢复被覆盖的代码－－－－－－－－－   
> :004044F9 669D                    popf   
> :004044FB 6661                    popa   
> :004044FD 6A00                    push 00000000   
>   
> \* Possible StringData Ref from Data Obj ->"Crackcode 2000 -- Author:Ru Feng "   
>                                         ->"(http:\\\\ocqpat.163.net)"   
>                                   |   
> :004044FF 68AC624000              push 004062AC   
> :00404504 E99ACFFFFF              jmp 004014A3   
>   
> 终于写完了，回头得让豆豆虾请我大餐不可！   
>   
> 【结束语】作家（Terry Berger)曾写道：“花园是生命本身的绝佳比喻。虽然并非   
> 所有人都是园丁，但我们都栽下家族和友谊的种籽，而的确，我们也必须悉心照   
> 顾它们，才能让它们生长茁壮。”不论你住在都市或乡村，独局或和他人合住，远   
> 离家人或是亲友就在身边---都请记住，你每天都要好好照顾自己的“花园”，必须付   
> 出时间、心血，和爱，照顾你的“部族”，才能得到快乐的酬报。   
>                                                                                     当当虾      耗时：8 小时   
>                                     2001 年 8 月 26 日 13:33  4dd466d0e1e0cc123afbe504da2462ca