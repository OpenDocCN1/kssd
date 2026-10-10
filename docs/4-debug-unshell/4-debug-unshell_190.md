

# Arm\_RestoreStolenCode1.0

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-573.htm`](https://www.pediy.com/kssd/pediy07/pediy7-573.htm)

已经加入的“回归”代码，代码布局小调整了一下  
  
暂定为：arm\_RestoreStolenCode1.0 ，附件下载  
  
由于我没有指令分析器，也不想做太复杂的工作，所以借助 OD 作为主体加载目标程序，OllyMachine 插件的脚本功能为我的 Dll 生成 “指令长度表” 和 “壳的 Jmp 表”（这个 Jmp 表在壳是有的，但不是我想要的，同时都不是每个人都愿意去跟这个表，于是我用脚本把它打印出来，运行上会有点慢，但可以接受），最后是用我的 Dll 的 RestorStolenCode 函数可以将 StolenCode 还原到 Code 节的相应位置！  
  
编程论坛的版主？“害我”，其实我发讨论贴是为了“偷懒”--希望有人接力，但正好被你搞垮了，我的成品还是半成品，你给我加了精，会容易引起别人误会，不得不让我加快脚步。。。  
  
由于我对这个壳不是深入了解，所以可能存在很多失误，如有失误之处还请指正！  
  
  
   为了清晰，同时我也是刚试手编程，代码方面没有优化，感觉格式方式还好，比从前的要看得舒服，不当之处还请见谅！  
  
   首先，你要看出壳填充的“垃圾”有什么规律  
  
1.分析垃圾的 种类 和 结合方式  
得出结果是：  
  
a.jmp 类跳转  
b.echg 类  
c.mov 类  
d.push pop 类  
e.not 类  
f.nop 类  
g.bswap 类  
  
其中，有些类是类中有分类的，功能一样的类长度也有分类，如 jmp 类，等等，我想你看到的比我说的要清楚明了...事实也是如此  
  
其次是明确我们做还原工作的原理和方向：（大致是先一个大循环（含小循环--指小块），处理大部分的垃圾，再就是做敏感的部分处理，最后是还原到 Code 节）  
  
这里我只说 push pop 类，因为大家都知道，堆栈的操作是很敏感的，要确保入栈出栈无误方能使程序运行有所保证，所以我这里独立分开一种 push pop “夹心饼” 的情况处理，而不是简单的检测配对，这里偷了个懒，没深入去找妥善的办法，循环 10 次就当处理好了，每次/小块只能保证清理一个！  
  
大体说一下用法：  
  
1.用 OD 载入目标程序（脱壳不脱壳都一样，就是要停在真正的 OEP），将 Dll 放在程序的同一目录。  
  
2.找出 StolenCode 节地址，这个你不要跟我说不会看，把程序的 .code 节和这个 StolenCode 节都设成可写权限，不确定可以点内存窗口找到相关的节，右键菜单点“完整权限”。  
  
3.填写两个参数：StolenCode 节起始地址 和 StolenCode 结束地址，运行 OllyMachine 脚本 Arm\_LogStolenCode\_Jmp\_DumpInfo.txt 获得两个表分别是：dwOpLenTab 和 dwLogAddressTab  
  
4. call dll 的 RestorStolenCode 函数（需要参数的）  
先调用 LoadLibraryA 加载和获取 dll 基址（成功加载地址在 eax）  
push "RestoreStolenCode.dll"  
call kernel32.LoadLibraryA  
  
再调用 GetProcAddress 获取函数地址（成功获取地址在 eax）  
push "RestoreStolenCode"  
call kernel32.GetProcAddress  
  
push dwStolenCodeSection   
push dwLogAddressTab  
push dwOpLenTab  
call eax  
test eax,eax  
if true is fixed, save .code section  
 else fail.  
  
by askformore  
6.11.2005  
  
Thanks you and me...  
  
  
\=================================================================================================  
// Tool: Ollydbg 1.10  and OllyMachine 2.0  
// Loading your target in Ollydbg, stop on real oep  
// Arm 的 StolenCode 记录数据脚本，这个脚本应该没有平台限制  
\_\_asm  
{  
pushfd  
pushad  
push 40  
push 1000  
push 20000      // 若 128K/2 都不能满足你 target 的 StolenCode，你可以加大这个申请空间  
push 0  
call KERNEL32.VirtualAlloc  
}  
  
cmp eax, 0  
je verr  
mov reg11, eax              // 这个申请空间的首地址是给 dwOpLenTab 的  
mov reg15, eax  
invoke logtext, "★★★★★ 成功申请下面的空间地址："  
invoke loglong, reg11  
invoke GotoDumpAddr, reg11  
  
\_\_asm  
{  
popad  
popfd  
}  
  
add reg15, 0x10000   // 这个是相对于申请空间的首地址，划分一个地址给  dwLogAddressTab 的  
mov reg10, 0x587000  // ☆☆☆ StolenCode 节首地址 ★★★ dwStolenCodeSection  
mov reg00, reg10  
  
mov reg12, 0  
mov reg13, 0  
mov reg14, reg11  
invoke GotoDumpAddr, reg11  
  
loop:  
cmp reg10, 0x596FC2  // ★★★ 此处是比较 StolenCode 节的结束地址 ☆☆☆  
je end  
  
mov reg16, reg10  
invoke ReadMemLong, reg16, 1  
cmp reg00, 0xE9  
jne skip  
inc reg16  
invoke ReadMemLong, reg16, 4  
cmp reg00, 0  
je skip  
dec reg16  
mov reg18, reg00  
invoke WriteMemLong, reg15, reg16, 4  
add reg15, 4  
add reg16, 5  
add reg16, reg18  // 取得返回地址  
  
checking:  
invoke GetPrevOpAddr, reg16, 1  // 逆向扫描返回地址  
mov reg16, reg00  
invoke ReadMemLong, reg16, 1  
cmp reg00, 0xE9  
jne checking                    // 直到找到 JMP 指令  
invoke WriteMemLong, reg15, reg16, 4  
add reg15, 4  
  
skip:  
invoke GetNextOpAddr, reg10, 1  
mov reg12, reg00  
sub reg12, reg10  
mov reg10, reg00  
invoke WriteMemLong, reg11, reg12, 1  
inc reg11  
inc reg13  
jmp loop  
  
end:  
invoke LogText, "☆☆☆☆☆记录指令数量为："  
invoke LogLong, reg13   
invoke DumpMem, reg14, 0x20000, "OpLen\_AddrTab.bin"  
invoke msg, "注意：Oplen\_AddrTab.bin 会生成在脚本目录，如不想生成文件可删除上一行脚本命令！"  
halt  
  
verr:  
\_\_asm  
{  
popad  
popfd  
}  
invoke msg, "发生一个申请内存的错误！"  
halt  

 4dd466d0e1e0cc123afbe504da2462ca