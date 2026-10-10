

# Rlpack 1.2x CodeReplace 修复方案

> 原文：[`www.pediy.com/kssd/pediy10/96831.html`](https://www.pediy.com/kssd/pediy10/96831.html)

关于 Rlpack 的 CodeReplace,我以前写过一个脚本,详见 UPK 脚本区,是逆向写的,也就是从加密过后的代码出发,找共同点,找特征,然后写脚本修复.效率低不说,而且容易漏,毕竟加壳后试炼品不一定存在所有类型的 Replace Code.并且由于在运算过程中,CPUID 还要参与计算,那么这个脚本要想跨平台,要么内联一段指令获取,要么就得重写,于是我选择了后者.值得一提的是,老外有个脚本叫做 VM Code Translater,显然他是理解有误,并且这个脚本运行效率也极低,所以我有必要重写一个.  
     于是我正向跟踪加壳流程,只选择 anticracking protection,以及 OPTIONS 里面的 Advanced AntiDump protection,数值稍微设置大点,其他的选项统统不选,这样可以确保在选择最小保护的前提下,依然让其 codereplace 功能生效.  
     OD 载入脱壳后的主程序,搜索字符串,查找到  
  

引用:   
Stolen instruction(s) number: %d  

  跟入前面的一个 CALL,下断选择试炼品,点保护,就中断下来了.这里便是 CodeReplace 出生的地方.  
  

引用:   
0040CDD8    55              push ebp  
0040CDD9    8BEC            mov ebp,esp  
0040CDDB    83C4 F8         add esp,-8  
0040CDDE    60              pushad  
0040CDDF    C705 CE164200 0>mov dword ptr ds:\[4216CE\],0  
0040CDE9    8B4D 0C         mov ecx,dword ptr ss:\[ebp+C\]  
0040CDEC    8B7D 08         mov edi,dword ptr ss:\[ebp+8\]  
0040CDEF    8B75 1C         mov esi,dword ptr ss:\[ebp+1C\]  
0040CDF2    8B45 18         mov eax,dword ptr ss:\[ebp+18\]  
0040CDF5    0345 14         add eax,dword ptr ss:\[ebp+14\]  
0040CDF8    8945 18         mov dword ptr ss:\[ebp+18\],eax  
0040CDFB    E9 2D0B0000     jmp 0040D92D                             ; 0040D92D  
0040CE00    803F 55         cmp byte ptr ds:\[edi\],55  
0040CE03    75 0C           jnz short 0040CE11                       ; 0040CE11  

  通过跟踪这个 CALL 我知道 Rlpack 1.2x CodeReplace 总共有 31 种形式的变形 分别为  

引用:   
1.push xx            68  
2.call \[\]               15FF  
3.mov \[\],eax        A3  
4.mov \[\],ecx        0D89  
5.mov \[\],edx        1589  
6.mov \[\],edi         3D89  
7.mov ecx,\[\]        0D8B  
8.push \[\]             35FF  
9.mov \[\],esi          3589  
A.CMP \[\],EBX        1D39  
B.CMP \[\],EAX        0539  
C.CMP \[\],ECX        0D39  
D.CMP \[\],EDX        1539  
E.CMP \[\],ESI         3539  
F.CMP \[\],EDI         3D39  
10.MOV EAX,\[\]      0A1  
11.MOV EAX,XX     0B8  
12.MOV EBX,XX     0BB  
13.MOV ECX,XX     0B9  
14.MOV EDX,XX     0BA  
15.CALL XX           0E8  
16.MOV EBX,\[\]      1D8B  
17.MOV EDX,\[\]      158B  
18.MOV ESI,\[\]       358B  
19.MOV EDI,\[\]       3D8B  
1A.ADD EAX,XX      05  
1B.SUB EAX,XX        2D  
1C.XOR EAX,XX       35  
1D.OR EAX,XX         0D  
1E.ADD EBX,XX        0C381  
1F.SUB EBX,XX        0EB81  
20.XOR EBX,XX        0F381  
21.OR EBX,XX          0CB81  
22.ADD ECX,XX        0C181  
23.SUB ECX,XX        0E981  
24.XOR ECX,XX        0F181  
25.OR ECX,XX          0C981  
26.ADD EDX,XX       0C281  
27.SUB EDX,XX       0EA81  
28.XOR EDX,XX       0F281  
29.OR EDX,XX        0CA81  
2A.ADD ESI,XX       0C681  
2B.SUB ESI,XX        0EE81  
2C.XOR ESI,XX       0F681  
2D.OR ESI,XX        0CE81  
2E.ADD EDI,XX      0C781  
2F.SUB EDI,XX       0EF81  
30.XOR EDI,XX      0F781  
31.OR EDI,XX        0CF81  

分别为编号,操作指令,机器码.在加壳过后的程序中对应的一张表 保存着类似这样的信息  

引用:   
003D0000  42 1A 00 00 01 00 00 00 F2 1A 00 00 4D 1A 00 00  B.....?..M..  
003D0010  01 00 00 00 C8 65 05 00 52 1A 00 00 15 00 00 00  ....R.....  
003D0020  90 13 00 00 60 1A 00 00 01 00 00 00 C8 65 05 00  ?..\`......  
003D0030  65 1A 00 00 15 00 00 00 98 13 00 00 6F 1A 00 00  e.....?..o..  
003D0040  15 00 00 00 00 14 00 00 79 1A 00 00 15 00 00 00  ......y.....  
003D0050  00 14 00 00 83 1A 00 00 15 00 00 00 00 14 00 00  ...?........  
003D0060  8F 1A 00 00 15 00 00 00 70 13 00 00 94 1A 00 00  ?.....p..?..  
003D0070  03 00 00 00 20 66 05 00 A2 1A 00 00 11 00 00 00  ... f.?.....  
003D0080  03 00 00 00 A7 1A 00 00 17 00 00 00 20 66 05 00  ...?..... f.  
003D0090  C5 1A 00 00 03 00 00 00 14 66 05 00 D9 1A 00 00  ?.....f.?..  
003D00A0  01 00 00 00 F9 1A 00 00 E7 1A 00 00 01 00 00 00  ...?..?.....  
003D00B0  C8 65 05 00 EC 1A 00 00 15 00 00 00 A0 13 00 00  .?.....?..  
003D00C0  14 1B 00 00 01 00 00 00 D6 1B 00 00 28 1B 00 00  .....?..(..  

这张表每个成员的大小为 0C 个字节,每个成员中有 3 个组成部分,分别为原始代码地址(减去基址后的),操作类型(上面的 31 种中的某一种),指令数  
  
为了解释上面的结论,我们还是来正向看他的加密流程.有 31 种情况的判断,我这里摘取第一种类型来分析  
  

引用:   
0040CE6F    803F 68         cmp byte ptr ds:\[edi\],68 //edi 指向试炼品代码段中某条指令地址   
0040CE72    75 37           jnz short 0040CEAB                   //是否为 PUSH,不相等则跳到下一个操作类型判断  
0040CE74    83F8 05         cmp eax,5                              //比较指令长度是否为 5.不相等则跳到下一个操作类型判断  
0040CE77    75 32           jnz short 0040CEAB                      
0040CE79    8B5F 01         mov ebx,dword ptr ds:\[edi+1\]   //取 PUSH 之后操作数大小      
0040CE7C    3B5D 14         cmp ebx,dword ptr ss:\[ebp+14\]//比较是否小于试炼品的基址  
0040CE7F    0F82 540A0000   jb 0040D8D9                       //如果小于则壳不 Replace,跳回循环  
0040CE85    3B5D 18         cmp ebx,dword ptr ss:\[ebp+18\]//比较是否大于镜像基址+镜像大小  
0040CE88    0F87 4B0A0000   ja 0040D8D9                      //大于则不处理  
0040CE8E    8B55 FC         mov edx,dword ptr ss:\[ebp-4\]  //取当前指令映射到内存中对应的内存地址(这里已经减去了基址了)  
0040CE91    8916            mov dword ptr ds:\[esi\],edx      //写入到表的第一个 DWORD 里  
0040CE93    8B57 01         mov edx,dword ptr ds:\[edi+1\]//取操作数  
0040CE96    2B55 14         sub edx,dword ptr ss:\[ebp+14\]//减去基址  
0040CE99    C746 04 0100000>mov dword ptr ds:\[esi+4\],1//指令类型为 1,写入到第二个 DWORD 里  
0040CEA0    8956 08         mov dword ptr ds:\[esi+8\],edx  //把减去基址后的操作数写入到第三个 DWORD 里  
0040CEA3    83C6 0C         add esi,0C                             //表中成员大小 0C 个字节  
0040CEA6    E9 2E0A0000     jmp 0040D8D9                   //循环  

其他的类型与此基本一致,唯一需要注意的是,有一些指令,并不减去基址,比如 MOV 指令等.所以在修复的时候注意区别对待.  
  
那么这个时候我们可以写脚本来修复了,因为可能需要修复的地址太多,如果全部由脚本来完成修复,那么效率之低可想而知.看过我在脚本区发的 1.0 脚本带的视频的朋友可以看出来,脚本要跑很久.那么我们可以通过脚本配合 PATCH 的方法来提高效率.这种方法在 Volx 以及 fxyang 等众位前辈的脚本里面经常见到.于是写出如下 PATCH  
  

引用:   
@0x003B0000:  
pushad   
pushfd   
mov eax,0  
mov ecx,0  
sub eax,0C  
@loop:  
add eax,0C  
cmp dword\[eax\],0  
je @exit  
mov edx,dword\[eax+8\]  
add edx,ecx  
mov ebx,dword\[eax\]  
add ebx,ecx  
cmp dword\[eax+4\],2  
je @fix2  
cmp dword\[eax+4\],3  
je @fix3  
cmp dword\[eax+4\],4  
je @fix4  
cmp dword\[eax+4\],5  
je @fix5  
cmp dword\[eax+4\],6  
je @fix6  
cmp dword\[eax+4\],7  
je @fix7  
cmp dword\[eax+4\],8  
je @fix8  
cmp dword\[eax+4\],9  
je @fix9  
cmp dword\[eax+4\],A  
je @fixA  
cmp dword\[eax+4\],B  
je @fixB  
cmp dword\[eax+4\],C  
je @fixC  
cmp dword\[eax+4\],D  
je @fixD  
cmp dword\[eax+4\],E  
je @fixE  
cmp dword\[eax+4\],F  
je @fixF  
cmp dword\[eax+4\],10  
je @fix10  
cmp dword\[eax+4\],11  
je @fix11  
cmp dword\[eax+4\],12  
je @fix12  
cmp dword\[eax+4\],13  
je @fix13  
cmp dword\[eax+4\],14  
je @fix14  
cmp dword\[eax+4\],15  
je @fix15  
cmp dword\[eax+4\],16  
je @fix16  
cmp dword\[eax+4\],17  
je @fix17  
cmp dword\[eax+4\],18  
je @fix18  
cmp dword\[eax+4\],19  
je @fix19  
cmp dword\[eax+4\],1A  
je @fix1A  
cmp dword\[eax+4\],1B  
je @fix1B  
cmp dword\[eax+4\],1C  
je @fix1C  
cmp dword\[eax+4\],1D  
je @fix1D  
cmp dword\[eax+4\],1E  
je @fix1E  
cmp dword\[eax+4\],1F  
je @fix1F  
cmp dword\[eax+4\],20  
je @fix20  
cmp dword\[eax+4\],21  
je @fix21  
cmp dword\[eax+4\],22  
je @fix22  
cmp dword\[eax+4\],23  
je @fix23  
cmp dword\[eax+4\],24  
je @fix24  
cmp dword\[eax+4\],25  
je @fix25  
cmp dword\[eax+4\],26  
je @fix26  
cmp dword\[eax+4\],27  
je @fix27  
cmp dword\[eax+4\],28  
je @fix28  
cmp dword\[eax+4\],29  
je @fix29  
cmp dword\[eax+4\],2A  
je @fix2A  
cmp dword\[eax+4\],2B  
je @fix2B  
cmp dword\[eax+4\],2C  
je @fix2C  
cmp dword\[eax+4\],2D  
je @fix2D  
cmp dword\[eax+4\],2E  
je @fix2E  
cmp dword\[eax+4\],2F  
je @fix2F  
cmp dword\[eax+4\],30  
je @fix30  
cmp dword\[eax+4\],31  
je @fix31  
  
@fix1:  
  
mov byte\[ebx\],68  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix2:  
  
mov word\[ebx\],15FF  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix3:  
  
mov byte\[ebx\],0A3  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix4:  
  
mov word\[ebx\],0D89  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix5:  
  
mov word\[ebx\],1589  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix6:  
  
mov word\[ebx\],3D89  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix7:  
  
mov word\[ebx\],0D8B  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix8:  
  
mov word\[ebx\],35FF  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix9:  
  
mov word\[ebx\],3589  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fixA:  
  
mov word\[ebx\],1D39  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fixB:  
  
mov word\[ebx\],0539  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fixC:  
  
mov word\[ebx\],0D39  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fixD:  
  
mov word\[ebx\],1539  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fixE:  
  
mov word\[ebx\],3539  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fixF:  
  
mov word\[ebx\],3D39  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix10:  
  
mov byte\[ebx\],0A1  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix11:  
sub edx,ecx  
mov byte\[ebx\],0B8  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix12:  
sub edx,ecx  
mov byte\[ebx\],0BB  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix13:  
sub edx,ecx  
mov byte\[ebx\],0B9  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix14:  
sub edx,ecx  
mov byte\[ebx\],0BA  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix15:  
  
mov byte\[ebx\],0E8  
sub edx,ebx  
sub edx,5  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix16:  
  
mov word\[ebx\],1D8B  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix17:  
  
mov word\[ebx\],158B  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix18:  
  
mov word\[ebx\],358B  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix19:  
  
mov word\[ebx\],3D8B  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix1A:  
sub edx,ecx  
mov byte\[ebx\],05  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix1B:  
sub edx,ecx  
mov byte\[ebx\],2D  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix1C:  
sub edx,ecx  
mov byte\[ebx\],35  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix1D:  
sub edx,ecx  
mov byte\[ebx\],0D  
mov dword\[ebx+1\],edx  
jmp @loop  
  
@fix1E:  
sub edx,ecx  
mov word\[ebx\],0C381  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix1F:  
sub edx,ecx  
mov word\[ebx\],0EB81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix20:  
sub edx,ecx  
mov word\[ebx\],0F381  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix21:  
sub edx,ecx  
mov word\[ebx\],0CB81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix22:  
sub edx,ecx  
mov word\[ebx\],0C181  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix23:  
sub edx,ecx  
mov word\[ebx\],0E981  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix24:  
sub edx,ecx  
mov word\[ebx\],0F181  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix25:  
sub edx,ecx  
mov word\[ebx\],0C981  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix26:  
sub edx,ecx  
mov word\[ebx\],0C281  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix27:  
sub edx,ecx  
mov word\[ebx\],0EA81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix28:  
sub edx,ecx  
mov word\[ebx\],0F281  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix29:  
sub edx,ecx  
mov word\[ebx\],0CA81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix2A:  
sub edx,ecx  
mov word\[ebx\],0C681  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix2B:  
sub edx,ecx  
mov word\[ebx\],0EE81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix2C:  
sub edx,ecx  
mov word\[ebx\],0F681  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix2D:  
sub edx,ecx  
mov word\[ebx\],0CE81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix2E:  
sub edx,ecx  
mov word\[ebx\],0C781  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix2F:  
sub edx,ecx  
mov word\[ebx\],0EF81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix30:  
sub edx,ecx  
mov word\[ebx\],0F781  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@fix31:  
sub edx,ecx  
mov word\[ebx\],0CF81  
mov dword\[ebx+2\],edx  
jmp @loop  
  
@exit:  
popfd  
popad  
nop  
nop  
nop  
nop  
nop  

思路就是按照他的加密流程,逆着推算回去,里面的  
  

引用:   
@0x003B0000:  
mov eax,0  
mov ecx,0  

003B0000 是申请的内存地址,这个是为后续的用 SKYPATCH 提取二进制码而写的,各位自行修改即可.在后面这个值并不需要.  
eax 这里是一个需要注意的地方,因为我在着手写全自动全保护脱壳脚本,考虑到大家试炼品不大一样,勾选的保护选项也不尽相同,所以我暂时没必要写成自动获取的,仅仅通过一个 ASK 来让各位输入 CodeReplace 表的地址.详情见附件中附带的视频.  
ecx 保存的是试炼品的基址  
  
然后由脚本来完成申请内存,获取基址,初始化环境和分配运行流程的任务即可.详见附件脚本.  
  
这样一来,修复 CodeReplace 就在瞬间精确完成了.如果脚本出错,修改 PE 头和所有区段的访问为全部访问即可.  
  
PS:附件传不上来 给网盘链接好了  
http://www.rayfile.com/files/1de9fa8a-9605-11de-a029-0014221b798a/