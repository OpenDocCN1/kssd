
# [原创]UPX2.92b分析和静态脱壳机(附源码)

自己用汇编写一个HelloWorld,用UPX2.92B压缩后,IDA载入分析

入口点如下:

```
UPX1:00406120 start proc near

UPX1:00406120

UPX1:00406120 var_AC= byte ptr -0ACh

UPX1:00406120

UPX1:00406120     pusha

UPX1:00406121     mov esi, offset dword_406000        ; upx1 Section UPX1区段,入口点所在,保存着被压缩程序的信息和压缩后的代码

UPX1:00406121                                         ; 接下来的解压代码要做的事就是找到被压缩程序的代码并把他们还原回去

UPX1:00406126     lea edi, [esi-5000h]                ; UPX0->VirtualSize == 5000h ,得到首区段的起始地址,被压缩的程序将恢复到这个区段

UPX1:0040612C     push edi                            ; 保存第一个区段的RVA

UPX1:0040612D     or  ebp, 0FFFFFFFFh

UPX1:00406130     jmp short loc_406142                ; 开始解压代码

UPX1:00406132 ; ---------------------------------------------------------------------------

UPX1:00406132     nop

```

下面就是壳的解压代码了,转成C代码并不太难,主要是一些关于标志位的运算有点绕人,如下面一段:

00406184   .  8B1E          mov     ebx, dword ptr ds:[esi]
00406186   .  83EE FC       sub     esi, -4
00406189   .  11DB          adc     ebx, ebx

`sub esi,-4`会改变CF标志位,而接下来就有一句跟标志位有关的加法运算,而且这一段在解压代码中多次出现

我定义了一个unsigned `__int64` `tmp`变量

`tmp=esi;`
`tmp-=(unsigned int)-4;`
`int nCF=((tmp >> 32) & 1);`

这样就可以得到CF标志位的值了

除了这一段,其他都是基本的逻辑,应该没什么问题

下面是壳的Shell代码中对于CALL指令的修正:

```
UPX1:004061F2 loc_4061F2:                             ; CODE XREF: start+5Cj

UPX1:004061F2     pop esi                             ; esi==UPX0->VirtualAddress+ImageBase

UPX1:004061F3     mov edi, esi                        ; 准备开始做call指令修正

UPX1:004061F5     mov ecx, 3                          ; ecx做计数器,要查找3个E8,且后一个字节为00

UPX1:004061FA

UPX1:004061FA loc_4061FA:                             ; CODE XREF: start+E1j

UPX1:004061FA                                         ; start+E6j

UPX1:004061FA     mov al, [edi]

UPX1:004061FC     inc edi

UPX1:004061FD     sub al, 0E8h

UPX1:004061FF

UPX1:004061FF loc_4061FF:                             ; CODE XREF: start+104j

UPX1:004061FF     cmp al, 1

UPX1:00406201     ja  short loc_4061FA

UPX1:00406203     cmp byte ptr [edi], 0

UPX1:00406206     jnz short loc_4061FA

UPX1:00406208     mov eax, [edi]

UPX1:0040620A     mov bl, [edi+4]

UPX1:0040620D     shr ax, 8

UPX1:00406211     rol eax, 10h

UPX1:00406214     xchg al, ah                         ; 将eax的高位换到al

UPX1:00406216     sub eax, edi

UPX1:00406218     sub bl, 0E8h

UPX1:0040621B     add eax, esi                        ; 算出偏移,修正call的偏移量

UPX1:0040621D     mov [edi], eax                      ; 修复公式为,E8后的第4个字节,减去E8下一字节所在地址,

UPX1:0040621D                                         ; 再加上区段RVA,算出偏移量

UPX1:0040621F     add edi, 5

UPX1:00406222     mov al, bl

UPX1:00406224     loop loc_4061FF

```

接着是修复原PE的IAT:

```
UPX1:00406226     lea edi, [esi+4000h]                ; esi+4000保存了被压缩文件的导入函数名和PE头结构

UPX1:0040622C

UPX1:0040622C loc_40622C:                             ; CODE XREF: start+12Ej

UPX1:0040622C     mov eax, [edi]                      ; 得到第一个DWORD,保存了偏移用来计算地址

UPX1:0040622E     or  eax, eax

UPX1:00406230     jz  short loc_40626E                ; 导入函数是否全部完成

UPX1:00406232     mov ebx, [edi+4]                    ; 得到第二个DWORD,用这个DWORD加上区段RVA得到被压缩文件

UPX1:00406232                                         ; 的IAT,后面得到的函数地址将写入这里

UPX1:00406235     lea eax, [eax+esi+6000h]            ; 定位到自身导入表的DLL名

UPX1:0040623C     add ebx, esi

UPX1:0040623E     push eax                            ; lpFileName

UPX1:0040623F     add edi, 8                          ; 偏移8个字节保存了IMAGE_IMPORT_BY_NAME

UPX1:00406242     call dword ptr [esi+603Ch]          ; call LoadLibraryA

UPX1:00406248     xchg eax, ebp                       ; ebp保存得到的HMODULE

UPX1:00406249

UPX1:00406249 loc_406249:                             ; CODE XREF: start+146j

UPX1:00406249     mov al, [edi]

UPX1:0040624B     inc edi                             ; 跳过Hint,来到Name

UPX1:0040624C     or  al, al

UPX1:0040624E     jz  short loc_40622C                ; 得到第一个DWORD,保存了偏移用来计算地址

UPX1:00406250     mov ecx, edi

UPX1:00406252     push edi                            ; lpProcName

UPX1:00406253     dec eax

UPX1:00406254     repne scasb                         ; 取得下一个函数名的地址

UPX1:00406256     push ebp                            ; hModule

UPX1:00406257     call dword ptr [esi+6040h]          ; call GetProcAddress

UPX1:0040625D     or  eax, eax                        ; 检查函数是否成功

UPX1:0040625F     jz  short loc_406268                ; 不成功跳向ExitProcess

UPX1:00406261     mov [ebx], eax                      ; 将函数地址写入被压缩文件的IAT

UPX1:00406263     add ebx, 4                          ; +4,准备写下一个

UPX1:00406266     jmp short loc_406249

UPX1:00406268 ; ---------------------------------------------------------------------------

UPX1:00406268

UPX1:00406268 loc_406268:                             ; CODE XREF: start+13Fj

UPX1:00406268     call dword ptr [esi+6050h]          ; call ExitProcess

UPX1:0040626E

UPX1:0040626E loc_40626E:                             ; CODE XREF: start+110j

UPX1:0040626E     mov ebp, [esi+6044h]

UPX1:00406274     lea edi, [esi-1000h]

UPX1:0040627A     mov ebx, 1000h

UPX1:0040627F     push eax

UPX1:00406280     push esp

UPX1:00406281     push 4

UPX1:00406283     push ebx

UPX1:00406284     push edi

UPX1:00406285     call ebp                            ; call VirtualProtct 修改Header为可写

UPX1:00406285                                         ; 用来修正区段Characteristics

UPX1:00406287     lea eax, [edi+1CFh]

UPX1:0040628D     and byte ptr [eax], 7Fh

UPX1:00406290     and byte ptr [eax+28h], 7Fh

UPX1:00406294     pop eax

UPX1:00406295     push eax

UPX1:00406296     push esp

UPX1:00406297     push eax

UPX1:00406298     push ebx

UPX1:00406299     push edi

UPX1:0040629A     call ebp                            ; 还原后壳的工作完成,做完收尾和清扫工作后跳向OEP

UPX1:0040629C     pop eax

UPX1:0040629D     popa

UPX1:0040629E     lea eax, [esp+2Ch+var_AC]

UPX1:004062A2

UPX1:004062A2 loc_4062A2:                             ; CODE XREF: start+186j

UPX1:004062A2     push 0

UPX1:004062A4     cmp esp, eax

UPX1:004062A6     jnz short loc_4062A2

UPX1:004062A8     sub esp, 0FFFFFF80h

UPX1:004062AB     jmp near ptr dword_401000

```

静态脱壳机的编写思路:

1. 将整个压缩PE根据每个区段的VirtualAddress映射到自己申请的堆空间
2. 根据壳的Shell代码自身的解压算法来解码被压缩的PE
3. 修正OEP和各区段的SizeOfRawData和PointerToRawData
4. 将堆空间中的代码和数据重新组装成一个PE文件,即为脱壳后的文件

附件是静态脱壳机源代码

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [UnpackUPX.rar](javascript:void(0);)

  （155.38kb，299次下载）
