

# JDPack1.01 加壳文件的反单步调试分析

> 原文：[`www.pediy.com/kssd/pediy10/83506.html`](https://www.pediy.com/kssd/pediy10/83506.html)

文章标题】: JDPack1.01 加壳文件的反单步调试分析  
【文章作者】: zyqex  
【作者邮箱】: zyqex@163.com  
【软件名称】: JDPack  
【加壳方式】: JDPackv1.01  
【编写语言】: win32 asm  
【使用工具】: fi、OD  
【操作平台】: xp2  
【作者声明】: 在学习 weiyi75\[Dfcg\] 的《手动脱壳入门第八篇 JDPack1.01》时的一点小发现，内容有些陈旧，可能早  
  
有人写过，菜鸟献丑了！  
\--------------------------------------------------------------------------------  
【详细过程】  
  注：原文是用单步跟踪法寻找 OEP  
  【Begin】:  
    
  fi 查壳结果：JDPack v1.01  
  od 载入 JDPack 安装后主文件，下面引用原文的部分内容：  
  ------------------------------------------------------------------  
  0040E000 >  60              PUSHAD  开始地点，注意 PUSHAD 关键字  
  0040E001    E8 00000000     CALL JDPACK.0040E006 F7 步过，变形 Jmp  
  0040E006    5D              POP EBP  
  0040E007    8BD5            MOV EDX,EBP  
  0040E009    81ED C62B4000   SUB EBP,JDPACK.00402BC6  
  0040E00F    2B95 3D344000   SUB EDX,DWORD PTR SS:\[EBP+40343D\]  
  0040E015    81EA 06000000   SUB EDX,6  
  0040E01B    8995 41344000   MOV DWORD PTR SS:\[EBP+403441\],EDX  
  0040E021    83BD 45344000 0>CMP DWORD PTR SS:\[EBP+403445\],0  
  0040E028    0F85 BC030000   JNZ JDPACK.0040E3EA  不跳。  
  0040E02E    C785 45344000 0>MOV DWORD PTR SS:\[EBP+403445\],1  
  0040E038    B9 64070000     MOV ECX,764  
  0040E03D    8DB5 182C4000   LEA ESI,DWORD PTR SS:\[EBP+402C18\]  
  0040E043    8A85 3C344000   MOV AL,BYTE PTR SS:\[EBP+40343C\]  
  0040E049    8A1E            MOV BL,BYTE PTR DS:\[ESI\]  
  0040E04B    32C3            XOR AL,BL  
  0040E04D    8806            MOV BYTE PTR DS:\[ESI\],AL  
  0040E04F    889D 3C344000   MOV BYTE PTR SS:\[EBP+40343C\],BL  
  0040E055    46              INC ESI  
  0040E056  ^ E2 EB           LOOPD SHORT JDPACK.0040E043 往回的循环结构,是程序在内存中解压过程。  
  0040E058    9C              PUSHFD  F4 到这里发现语句有了一些变化。  
  0040E059    E7 11           OUT 11,EAX                               ; I/O 命令  
  0040E05B    D5 D4           AAD 0D4  
  0040E05D    A0 A727922D     MOV AL,BYTE PTR DS:\[2D9227A7\]  
  0040E062    0242 42         ADD AL,BYTE PTR DS:\[EDX+42\]  
  0040E065    BD 36836A5B     MOV EBP,5B6A8336  
  ...........................................................  
  0040E058    9C              PUSHFD    
  0040E059    58              POP EAX  
  0040E05A    F6C4 01         TEST AH,1  
  0040E05D    74 07           JE SHORT JDPACK.0040E066 隔壁也跳。  
  0040E05F    80B5 BF2F4000 F>XOR BYTE PTR SS:\[EBP+402FBF\],0FF  
  0040E066    8BB5 E9314000   MOV ESI,DWORD PTR SS:\[EBP+4031E9\]  跳到这里  
  0040E06C    8BC5            MOV EAX,EBP  
  0040E06E    56              PUSH ESI  
  0040E06F    50              PUSH EAX  
  0040E070    8B88 F1314000   MOV ECX,DWORD PTR DS:\[EAX+4031F1\]  
  0040E076    6A 04           PUSH 4  
  0040E078    68 00100000     PUSH 1000  
  0040E07D    51              PUSH ECX  
  0040E07E    6A 00           PUSH 0  
  0040E080    FF95 C8334000   CALL DWORD PTR SS:\[EBP+4033C8\]  
  ---------------------------------------------------------------------------------  
  注意 0040E05D    74 07           JE SHORT JDPACK.0040E066 处，我在依照原文步骤运行到此处时，  
  OD 显示“跳转未实现”，如果继续下去不久程序终止。于是，重载程序，运行到 0040E05A 处时改 Z 标志位，  
  让它跳，然后根据原文就可找到 OEP.  
  现在，来看看这里  
  0040E058    9C              PUSHFD    
  0040E059    58              POP EAX  
  0040E05A    F6C4 01         TEST AH,1  
  0040E05D    74 07           JE SHORT JDPACK.0040E066 隔壁也跳。  
  很明显这是在测试 Eflags 标志寄存器里的 TF 位是否为 1，intel software developer's manual(basic architecture)里  
  如下描述 TF 位：  
  Trap flag - Set to enable single-step mode for debugging;clear to disable single-step mode  
  大体来说程序就是利用测试 TF 位判是否单步调试，如果被单步调试就终止程序，这样就起到了一个简单的反单步调  
  
试  
  的效果。可能有人会想到直接将 0040E05D 处 JE patch 成 JMP，现在看看下面这几句就知道行不行得通了：  
  0040E038    B9 64070000     MOV ECX,764  
  0040E03D    8DB5 182C4000   LEA ESI,DWORD PTR SS:\[EBP+402C18\]  
  0040E043    8A85 3C344000   MOV AL,BYTE PTR SS:\[EBP+40343C\]  
  0040E049    8A1E            MOV BL,BYTE PTR DS:\[ESI\]  
  0040E04B    32C3            XOR AL,BL  
  0040E04D    8806            MOV BYTE PTR DS:\[ESI\],AL  
  0040E04F    889D 3C344000   MOV BYTE PTR SS:\[EBP+40343C\],BL  
  0040E055    46              INC ESI  
  0040E056  ^ E2 EB           LOOPD SHORT JDPACK.0040E043  
  这是在解码地址 0040E058 处开始的 0x764 个字节，所以 patch 的方法自然是不行的了。  
  上面的几句我用 c++编写代码模拟了其过程（代码在 vc6.0 里测试通过，为求简易分析，取解码长度为 5）  
    
  #include <iostream>  
  using namespace std;  
  int main()  
  {  
    int len=5;  
    int x=0x23;   
    int i,y;  
    int serial\[\]={0xBF, 0xE7 ,0x11 ,0xD5, 0xD4};    // 对应解码系列  9C， 58， F6， C4， 01，...         
  
               
    for(i=0;i<len;i++)  
      {  
        y=serial\[i\];  
          serial\[i\] = x ^ y;          //y 原码       serial\[i\]  XOR  x 可得原码  
            x=y;  
     }  
      cout<<hex<<x<<endl;  
    
    cout<<"解码系列为：";  
    hex(cout);              //十六进制输出  
    for(i=0;i<len;i++)  
      cout<<serial\[i\]<<'\\t';  
    
  //下面的注释部分是我写的一个逆算法，即根据解码后的内容反解码出原内容。原意是想让它反解码 jne(或 jmp),然后  
  
做  
  //patch,但是分析了一下算法中会根据前一个字节参与 xor 运算来解码当前字节，这样要  
//patch 0x764 个字节来去除反单步调试的功能，代价过大了，^\_^（内存补丁可以考虑考虑，不过目的是脱壳，所以直接修改 Z 位 enable 检测）  
    
   /\*(注释部分)  
    int len=5;  
    int x=0x22;  
    int i,y;  
    int serial\[\]={0x9C, 0x58, 0xF6, 0xC4, 0x01};       // 原码系列 0xBF, 0xE7 ,0x11 ,0xD5, 0xD4  
    for (i=0;i<len;i++)  
      {  
        y=serial\[i\];  
          serial\[i\] = y ^ x;            
        x=serial\[i\];  
            
      }  
    
    cout<<"原码系列为：";  
    hex(cout);  
    for(i=0;i<len;i++)  
      cout<<serial\[i\]<<'\\t';  
    
  \*/    
      
      
    return 0;  
  }  
  ----------------------------------------------------------------------------  
  【Over】  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  一点在前辈基础上的小发现，经验谈不上，希望对某些朋友有用。第一次写东西，文章不足之处望见谅，在下入门不久，菜鸟一个![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ，水平还有待提高！谢谢观赏(最后谢谢原文作者（weiyi75，不知是不是传说中的二哥![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑") ）还要谢谢 ctc editor 的作者，用这个东西写破文免去了不少麻烦)！  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 03 月 09 日 15:25:39