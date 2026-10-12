
`call <sub_5B1150>(in string [ebp-4], out string [ebp-8]),`

这才是关键重中之重, 前边都是为这里做铺垫的. 功能是: 把用户名 `qiweixue` 和机器码, 经过乱码成 `vosmiusa`, 再根据 `vosmiusa` 计算出

一个整数, 再把整数格式化成字符串, 再再提取出注册码: `1234567890123456789012` 的后四位, 也就是从 13 位开始提取, 最后相互比较,

如果相等就注册成功, 否则就 over. 至于注册码前 13 位有可能作者在程序限制功能中使用(纯属猜测). 分析如下:

```
005B1150  >/$  55          push    ebp
005B1151  |.   8BEC        mov     ebp, esp
005B1153  |.   83C4 E8     add     esp, -18
005B1156  |.   53          push    ebx
005B1157  |.   56          push    esi
005B1158  |.   33DB        xor     ebx, ebx
005B115A  |.   895D F4     mov     [ebp-C], ebx
005B115D  |.   895D F0     mov     [ebp-10], ebx
005B1160  |.   894D F8     mov     [ebp-8], ecx---->返回值地址
005B1163  |.   8955 FC     mov     [ebp-4], edx--->用户名:qiweixue
005B1166  |.   8B45 FC     mov     eax, [ebp-4]--->主窗体对象
005B1169  |.   E8 BE3FE5FF call    <@System@@LStrAddRef$qqrpv>--->引用用户名字符串
005B116E  |.   33C0        xor     eax, eax
005B1170  |.   55          push    ebp
005B1171  |.   68 47125B00 push    <sub_5B1247>
005B1176  |.   64:FF30     push    dword ptr fs:[eax]
005B1179  |.   64:8920     mov     fs:[eax], esp
005B117C  |.   8B45 FC     mov     eax, [ebp-4]--->--->用户名:qiweixue
005B117F  |.   E8 B83DE5FF call    <@@LStrLen_Or_DynArrayLength$qqrv>-->求长度函数
005B1184  |.   83F8 04     cmp     eax, 4
005B1187  |.   7D 0D       jge     short <loc_5B1196>->大于 4 就跳,否则 Over
005B1189  |.   8D45 F4     lea     eax, [ebp-C]
005B118C  |.   E8 EB3AE5FF call    <@System@@LStrClr$qqrpv>-->清返回地址为 0
005B1191  |.   E9 83000000 jmp     <loc_5B1219>
005B1196  |>|> 8D4D F0     lea     ecx, [ebp-10]
005B1199  |.   8B15 C0395C00 mov    edx, [5C39C0]  ---->机器码:76481-640-1153263-23182
005B119F  |.   8B12        mov     edx, [edx]
005B11A1  |.   8B45 FC     mov     eax, [ebp-4]--->用户名
005B11A4  |.   E8 6711F5FF call    <sub_502310>--->用户名和机器码相互处理
005B11A9  |.   8B45 F0     mov     eax, [ebp-10]-->得到结果:"vosmiusa"
005B11AC  |.   E8 8B3DE5FF call    <@@LStrLen_Or_DynArrayLength$qqrv>
005B11B1  |.   8BC8        mov     ecx, eax
005B11B3  |.   33DB        xor     ebx, ebx
005B11B5  |.   8BC1        mov     eax, ecx
005B11B7  |.   48          dec     eax
005B11B8  |.   85C0        test    eax, eax
005B11BA  |.   7C 14       jl      short <loc_5B11D0>--->如果小于 0 就跳
005B11BC  |.   40          inc     eax
005B11BD  |.   33D2        xor     edx, edx
005B11BF  |>|> 8B75 F0     /mov     esi, [ebp-10]
005B11C2  |.   0FB67416 FF |movzx   esi, byte ptr [esi+edx-1]--->逐次提取:vosmiusa
005B11C7  |.   0FAFF1      |imul    esi, ecx-->相互乘于其长度
005B11CA  |.   03DE        |add     ebx, esi--->相加
005B11CC  |.   42          |inc     edx
005B11CD  |.   48          |dec     eax
005B11CE  |.^ 75 EF        \jnz     short <loc_5B11BF>
005B11D0  |>|> 81FB 0F270000 cmp     ebx, 270F---------->计算结果与 270F 比较
005B11D6  |.   7E 0E       jle     short <loc_5B11E6>
005B11D8  |.   8BC3        mov     eax, ebx
005B11DA  |.   B9 10270000 mov     ecx, 2710---->加 2710
005B11DF  |.   99          cdq
005B11E0  |.   F7F9        idiv    ecx--->除以长度
005B11E2  |.   8BDA        mov     ebx, edx
005B11E4  |.   EB 0E       jmp     short <loc_5B11F4>
005B11E6  |>|> 81FB 28230000 cmp     ebx, 2328----------->结算结果与 2328 比较
005B11EC  |.   7D 06       jge     short <loc_5B11F4>
005B11EE  |.   81C3 E8030000 add     ebx, 3E8--->加 3E8
005B11F4  |>|> 8D45 F0     lea     eax, [ebp-10]
005B11F7  |.   50          push    eax    ---->返回值地址
005B11F8  |.   895D E8     mov     [ebp-18], ebx  ------->计算结果给[ebp-14]
005B11FB  |.   C645 EC 00  mov     byte ptr [ebp-14], 0
005B11FF  |.   8D55 E8     lea     edx, [ebp-18]
005B1202  |.   33C9        xor     ecx, ecx
005B1204  |.   B8 5C125B00 mov     eax, <off_5B125C> -->参数"%4d"
005B1209  |.   E8 AE9AE5FF call    <@Sysutils@Format$qqrx17System@Ansi>-->把整数格式为字符串
005B120E  |.   8D45 F4     lea     eax, [ebp-C]---->结果地址给 eax
005B1211  |.   8B55 F0     mov     edx, [ebp-10]
005B1214  |.   E8 FB3AE5FF call    <@System@@LStrLAsg$qqrpvpxv>-->字符串相互赋值
005B1219  |>|> 8B45 F8     mov     eax, [ebp-8]--->最终结果地址给 eax
005B121C  |.   8B55 F4     mov     edx, [ebp-C]
005B121F  |.   E8 AC3AE5FF call    <@System@@LStrAsg$qqrpvpxv>-->字符串相互赋值
005B1224  |.   33C0        xor     eax, eax
005B1226  |.   5A          pop     edx
005B1227  |.   59          pop     ecx
005B1228  |.   59          pop     ecx
005B1229  |.   64:8910     mov     fs:[eax], edx
005B122C  |.   68 4E125B00 push    <loc_5B124E>
005B1231  |>|> 8D45 F0     lea     eax, [ebp-10]
005B1234  |.   BA 02000000 mov     edx, 2
005B1239  |.   E8 623AE5FF call    <@System@@LStrArrayClr$qqrpvi>
005B123E  |.   8D45 FC     lea     eax, [ebp-4]
005B1241  |.   E8 363AE5FF call    <@System@@LStrClr$qqrpv>
005B1246  \\.  C3          retn
```

到这里基本流程分析完毕, 根据以上分析就可以写出一个注册机. 这个程序的分析的方法不外乎两种: 顺序分析: 根据程序启动的时候跟踪算法流程

其次是逆向分析: 根据注册窗口的事件, 和交互信息找出算法分析. 这里都可以的. 我是采用顺序分析, 因为这样写文章, 个人爱好吧!

还有一点必须说明: 就是输入注册窗体 `Edit` 控件中的机器码显示的是: `#93;89 739#6?0<2?>,5:??<#`, 而我这里说的是内存中的机器码 `76481-640-1153263-23182`,

`Edit` 控件显示的是乱码是迷惑我们, 当然可以解码就是用上边说的那个解码函数, 真正用的是内存中那个格式清晰的机器码.

## 计算本地机器码源代码

(C# .Net SDK 2.0 编译)

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using Microsoft.Win32;
using System.Diagnostics;

namespace MachinePediy
{
    class Pediy
    {
        static void Main(string[] args)
        {
            string userStr = "nEO iMAGING";
            string machineStr;
            char[] userCode;
            char[] machineCode;
            byte code1 = 15;
            byte code2 = 240;
            RegistryKey RegKey = Registry.LocalMachine;
            RegKey = RegKey.OpenSubKey("Software\\Microsoft\\Windows\\CurrentVersion");
            Object val = RegKey.GetValue("ProductId");
            // System.Console.WriteLine("Window OS ProductID:" + val);
            machineStr = ((String)val);
            userCode = userStr.ToCharArray();
            machineCode = machineStr.ToCharArray();
            for (int i = 0, j = 0; i < machineStr.Length; i++, j++)
            {
                byte a, b, c;
                if (j >= userStr.Length)
                {
                    j = 0;
                }
                a = (byte)machineCode[i];
                a = (byte)(a & code1);
                b = (byte)userCode[j];
                b = (byte)(b & code1);
                c = (byte)(a ^ b);
                a = (byte)machineCode[i];
                a = (byte)(a & code2);
                a = (byte)(a + c);
                machineCode[i] = (char)a;
            }
            System.Console.Write("机器码：" + "#");
            for (int i = 0; i < machineStr.Length; i++)
            {
                System.Console.Write(machineCode[i]);
            }
            System.Console.Write("#");
            System.Console.ReadLine();
        }
    }
}
```

## 计算注册码源代码

(C# .Net SDK 2.0 编译)

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using Microsoft.Win32;
using System.Diagnostics;

namespace Pediy
{
    class Pediy
    {
        static void Main(string[] args)
        {
            string machineStr;
            char[] machineCode;
            char[] userCode;
            byte code1 = 15;
            byte code2 = 240;
            Int32 key = 0;
            string keyStr = "76481-640-1153263-";
        }
    }
}
```
