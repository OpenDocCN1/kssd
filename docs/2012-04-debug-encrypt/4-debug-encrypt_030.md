
# 密码小游戏：暴力破解 CRC32 密码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-737.htm`](https://www.pediy.com/kssd/pediy07/pediy7-737.htm)

今天是 04 年最后一天了，回首往昔，感概万千。

发一篇以前整理的文章，附件包含 pdf 格式的文章、对象程序，程序为应一个朋友而破的一个东西。

注：文章仅为交流，有任何错误请指正。由于 `CRC32` 存在非常多的碰撞，我这里得出的密码并不能使程序正常往下执行，可能是还需要其它文章，或可能是我得的密码不对，由于时间不多，没有细看。本文仅为学习 `CRC32` 作的一个教程。

## 密码游戏：实例解说暴力破解 CRC32 密码的思路及实现

By lordor

**前置知识**：调试器 `ollydbg` 的简单使用，汇编基本知识，VC 编程的基本使用。
**对象**：电脑爱好者、软件破解爱好者、逆向工程爱好者
**工具**：`ollyDbg1.1` 英文版，`VC.NET`、`PEID0.92`、`Hex WorkShop4.23`
**目的**：找到密码，激活灰色按键。

说明：请打开附件带的程序 `PKProc_98.exe`，运行这个程序时要输入密码才能激活确定按键，现在我们就要找到密码，本文将介绍怎么找到程序的处理加密的过程，怎么用 VC 编写穷举程序进行密码的破解。

### 一、分析程序密码验证过程

先用 `PEID` 检测一下程序编写语言，显示：`Install Stub 32-bit -> InstallShield [Overlay]`，这样可以用 `OllyDbg` 直接反汇编，也可以动态跟踪。

这个程序的反汇编语句比较好读，流程清晰，有兴趣的朋友可以动态跟踪它的加载过程。这里我们用 API 断点法定位关键代码。由于程序是要密码正确才能激活确定按键，我们可以用 `EnableWindow` 下断。用 `OllyDbg` 载入程序，在命令行下输入 `bp EnableWindow` (注意字母大小写)，然后 `F9` 直接让程序运行起来，在密码框中附件输入一个字符，马上会中断在 `ollydbg` 中，按一下 `Alt+F9` 返回到程序的代码中，往上看就是程序的验证算法了。我们看一下这些代码：

```
100071EE CALL DWORD PTR DS:[<&USER32.GetDlgItemTe>; \GetDlgItemTextA
100071F4 LEA ECX,DWORD PTR SS:[EBP-200]                 //取得输入字符
100071FA PUSH ECX ; /String
100071FB CALL DWORD PTR DS:[<&KERNEL32.lstrlenA>]        //求字符长度
10007201 LEA EDX,DWORD PTR DS:[EAX+1]                   //字符长度加 1
10007204 LEA ECX,DWORD PTR SS:[EBP-200]                 //输入字符串地址入 ECX
1000720A CALL ginstall.10002F73                          //关键验证函数，得检验值入 EAX
1000720F MOV ECX,DWORD PTR DS:[1000BCA8]                //1000BCA8 处的值 0EE2AF34 赋给 ECX
10007215 SUB ECX,EAX                                    //执行减操作，这里改变 CF 标志
10007217 CMP ECX,1                                      //比较是否为 1
1000721A MOV ECX,3EF
1000721F SBB EAX,EAX                                    //相当于：EAX=EAX-(EAX+1)
10007221 NEG EAX                                        //相当于：EAX=-EAX
10007223 PUSH EAX                                        //压入 EAX
10007224 CALL ginstall.10006497                          //取得窗口 HWND 值
10007229 PUSH EAX ; |hWnd                               //压入 HWND
1000722A CALL DWORD PTR DS:[<&USER32.EnableWindow>;      //这里执行否要激活
10007230 POP ESI
```

大家可以看到这个过程比较清晰，也就是先取得输入的字符串，通过 `CALL ginstall.10002F73` 求这串字符串的 `CRC32` (后面我们会说)，然后与 `[1000BCA8]` 处的值比较，如果相等，则会执行 `EAX` 值是 `1` 还是 `0`，这个值就是 `EnableWindow` 的是否激活参数。我们再进 `CALL ginstall.10002F73` 看看这个 `CRC32` 算法，过程也不长，先记住执行这个 `CALL` 传入的两个参数，即 `EDX` 为字符串的长度，`ECX` 为字符的地址：

```
10002F73   PUSH ESI         //保存
10002F74   MOV EAX,-1       //EAX=FFFFFFFF
10002F79   PUSH EDI         //保存
10002F7A   MOV ESI,EDX      //字符串长度入 ESI
10002F7C   ADD EDX,EAX      //EDX=EDX+EAX，长度减 1
10002F7E   TEST ESI,ESI     //看 ESI 是否为 0，即是否有字符
10002F80   JE SHORT ginstall.10002FA6  //为 0 则直接跳走
10002F82   /MOVZX ESI,BYTE PTR DS:[ECX]  //取一位字符入 ESI
10002F85   |MOVZX EDI,AX    //EDI=AX
10002F88   |SHR EAX,8       //EAX 右移 8 位
10002F8B   |XOR ESI,EDI     //EDI 与一位字符作异或运算
10002F8D   |AND ESI,0FF     //ESI 取低 8 位
10002F93   |INC ECX         //字符地址加一
10002F94   |MOV ESI,DWORD PTR DS:[ESI*4+1000EBE0] //以 1000EBE0 为基址，读入 4 位字节
10002F9B   |XOR ESI,EAX     //ESI=ESI XOR EAX
10002F9D   |MOV EAX,ESI     //保存 ESI 到 EAX 中
10002F9F   |MOV ESI,EDX     //ESI=EDX
10002FA1   |DEC EDX         //长度减一
10002FA2   |TEST ESI,ESI    //长度是否为 0
10002FA4   \JNZ SHORT ginstall.10002F82  //字符长度不为 0，则继续循环
10002FA6   NOT EAX          //执行 NOT 操作
10002FA8   POP EDI
10002FA9   POP ESI
10002FAA   RETN
```

这个 `Call` 是一个变形的 `CRC32`，`1000EBE0` 处存放的是 `CRC32` 用到的 256 个 4 个字节长的表，其表值与标准的 `CRC32` 表值不一样，在后面我也看到，作者修改的这个表值使得 `CRC32` 算法安全性大大降低。更详细的 `CRC32` 算法说明请大家参考相关的资料。

### 二、分离出主要验证关键函数

从上面的分析，我们已经清楚程序的验证过程了。由于 `CRC32` 属于散列值，我们没方法据 `CRC32` 的值求得原先的值，所以必须穷举所有可能的密码串的 `CRC32` 值，比较是否与预定的值相等。为了更可靠的穷举，我们要从以上分析的代码中生成我们的关键函数：求 `CRC32` 算法。

首先可以把这段汇编代码拷贝出来，只要在 VC 中嵌入汇编代码就可以使用了。其次因为 `1000EBE0` 处存放着表，所以必须要把 `1000EBE0` 处的内容拷贝出来，我是用 `IsDebuggerPresent` 插件中的 `DUMP` 工具保存下来的。为什是 400 的长度？因为 `256*4=1024` 个字节，转换为十六进制则为 `400`。然后用 `Hex WorkShop` 打开 dump 出来的文件，先择所有的字节，再点击“EDIT/Copy As/C Source”,再在 VC 的文件中粘贴，就会生成适合 VC 使用的数据了。最后构成关键代码如下：

```c
//功能：求输入字符串的 crc32 值
//参数：name 为字符串地址，len 为字符串的长度
//返回：CRC32 值
DWORD GetCrc(char* name, size_t len) {
    //edx=字符串的长度
    //ecx=字符串地址
    DWORD dwcrc;
    __asm{
        mov edx, len;
        mov ecx, name[0];
        //PUSH ESI;
        MOV EAX,-1;
        //PUSH EDI;
        MOV ESI,EDX;
        ADD EDX,EAX;
        TEST ESI,ESI;
        JE SHORT _10002FA6;
    _10002F82:
        MOVZX ESI,BYTE PTR DS:[ECX];
        MOVZX EDI,AX;
        SHR EAX,8;
        XOR ESI,EDI;
        AND ESI,0xFF;
        INC ECX;
        MOV ESI,DWORD PTR DS:[ESI*4+basetable];
        XOR ESI,EAX;
        MOV EAX,ESI;
        MOV ESI,EDX;
        DEC EDX;
        TEST ESI,ESI;
        JNZ SHORT _10002F82;
    _10002FA6:
        NOT EAX ;==>必须等于 eax=0EE2AF34
        mov dwcrc,eax;
        //POP EDI;
        //POP ESI;
    }
    if(dwcrc==0x0EE2AF34) {
        char buff[256];
        wsprintf(buff,"Find the Key:%s",name);
        MessageBoxA(NULL,buff,"Succeed!",MB_OK|MB_TOPMOST);
    }
    return dwcrc;
}
```

说明：其中 `basetable` 定义为数组：`unsigned char basetable[1024]`，内容就是上面拷贝出来的数据；`0x0EE2AF34` 为程序中正确的密码的 `CRC32` 值。

### 三、设计我们的穷举程序

有了前面的分析，后面的工作就简单了，现在我们来穷举长度为 10 位以内的密码，编写代码如下。下面的代码只是为了方便说明，没有采用其它算法，也没有作算法及代码的优化。

```c
void CbrutforceDlg::OnBnClickedBegin() {
    // TODO: Add your control notification handler code here
    char BaseCode[]="0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"; //可见的字符 36
    size_t BaseCodeLen=strlen(BaseCode);  //长度
    char PassWord[256];  //定义测试的密码串，并初始化为 0
    memset(PassWord,0x0,256);
    size_t a1,a2,a3,a4,a5,a6,a7,a8,a9,a10;  //定义密码下标位
    for(a1=0;a1<BaseCodeLen;a1++)  //这里穷举 10 位的密码
    {
        //求长度为 1 位的密码
        PassWord[0]=BaseCode[a1];
        PassWord[1]=0x0;
        GetCrc(PassWord,2);
        for(a2=0;a2<BaseCodeLen;a2++) {
            //求长度为 2 位的密码
            PassWord[1]=BaseCode[a2];
            PassWord[2]=0x0;
            GetCrc(PassWord,3);
            for(a3=0;a3<BaseCodeLen;a3++) {
                //求长度为 3 位的密码
                PassWord[2]=BaseCode[a3];
                PassWord[3]=0x0;
                GetCrc(PassWord,4);
                for(a4=0;a4<BaseCodeLen;a4++) {
                    //求长度为 4 位的密码
                    PassWord[3]=BaseCode[a4];
                    PassWord[4]=0x0;
                    GetCrc(PassWord,5);
                    for(a5=0;a5<BaseCodeLen;a5++) {
                        //求长度为 5 位的密码
                        PassWord[4]=BaseCode[a5];
                        PassWord[5]=0x0;
                        GetCrc(PassWord,6);
                        for(a6=0;a6<BaseCodeLen;a6++) {
                            //求长度为 6 位的密码
                            PassWord[5]=BaseCode[a6];
                            PassWord[6]=0x0;
                            GetCrc(PassWord,7);
                            for(a7=0;a7<BaseCodeLen;a7++) {
                                //求长度为 7 位的密码
                                PassWord[6]=BaseCode[a7];
                                PassWord[7]=0x0;
                                GetCrc(PassWord,8);
                                for(a8=0;a8<BaseCodeLen;a8++) {
                                    //求长度为 8 位的密码
                                    PassWord[7]=BaseCode[a8];
                                    PassWord[8]=0x0;
                                    GetCrc(PassWord,9);
                                    for(a9=0;a9<BaseCodeLen;a9++) {
                                        //求长度为 9 位的密码
                                        PassWord[8]=BaseCode[a9];
                                        PassWord[9]=0x0;
                                        GetCrc(PassWord,10);
```



```
for(a10=0;a10<BaseCodeLen;a10++) {
  //求长度为 10 位的密码
  PassWord[9]=BaseCode[a10];
  PassWord[10]=0x0;
  GetCrc(PassWord,11);
}
```

`AfxMessageBox("Finish!");`

算法说明：定义所有的可见字符、测试的密码串及密码串的下标，过程就是通过循环来穷举。详细的请看源代码。

现在开始编译一下程序，运行程序就开始进行密码穷举了。在我的机器 AMD Athlon 1.8G+512M DDR333 的机器下，十几分钟就会找到密码。按确定后还可以继续查找下一个密码，由于这个算法存在的漏洞，半个小时内会找到好几个。注意：由于穷举很耗系统资源，运行程序后，`CPU` 会占用 100%。看看成功的画面.

总结：对于这个分析的过程来说，首先要找到处理密码的地方，并找出加密判断的地方，如本例找到 `CRC32` 加密后值，再后模拟它的过程，对密码进行暴力破解。这种方法可能推广使用到其它地方，如 `WIPZIP/WINRAR` 加密码，`Word` 文档加密码，还有如 `Md5` 的暴力破解等场合。对于要破解的密码来说，关键是要找到要处理密码的函数，以及判断密码是否正确的地方，而穷举过程来说，关键是对穷举算法的设计，达到缩短穷举算法的时间。

不过对于穷举来说，目前还没有一个很好的思路来设计一个穷举算法。如果大家有好的建议请发邮件给我

关于我：网名：`lordor`

Mail：`lordor@163.com`

QQ:88378557

介绍：自由撰稿人。喜欢系统底层、密码学、程序逆向分析及软件加密。目前兴趣方向：嵌入式操作系统、移动操作平台等

于 04.12.31

`4dd466d0e1e0cc123afbe504da2462ca`
