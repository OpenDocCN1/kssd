
```assembly
:004AA368 3A4203          cmp al, byte ptr [edx+03]  
:004AA36B 0F8593030000    jne 004AA704  

:004AA371 8B45B0          mov eax, dword ptr [ebp-50]  
:004AA374 8A4005          mov al, byte ptr [eax+05]  
:004AA377 8B15C4ED4A00    mov edx, dword ptr [004AEDC4]  
:004AA37D 3A4205          cmp al, byte ptr [edx+05]  
:004AA380 0F857E030000    jne 004AA704  

:004AA386 8B45B0          mov eax, dword ptr [ebp-50]  
:004AA389 8A4001          mov al, byte ptr [eax+01]  
:004AA38C 8B15C4ED4A00    mov edx, dword ptr [004AEDC4]  
:004AA392 3A4201          cmp al, byte ptr [edx+01]  
:004AA395 0F8569030000    jne 004AA704  

:004AA39B 8B45B0          mov eax, dword ptr [ebp-50]  
:004AA39E 8A4007          mov al, byte ptr [eax+07]  
:004AA3A1 8B15C4ED4A00    mov edx, dword ptr [004AEDC4]  
:004AA3A7 3A4207          cmp al, byte ptr [edx+07]  
:004AA3AA 0F8580010000    jne 004AA530  
```

写注册机的算法是，把软件给的机器码前两位移到最后，然后是计算 2 处异或 80 次，再把结果按机器码每位的数字取出不同的数再异或 10 次（计算 1），结果就是注册码了。那 80 次的计算太烦了，我根据结果求出来实际上是与 `229898081` 异或，高手写个注册机吧。

软件中有许多乱七八糟的计算，好像是迷惑人的，因为没有仔细分析，失误之处还请各位大大指出。

---

### 标题
一个计算器的算法，前几天有人贴过简单的分析，这几天终于有时间了，当作复习就拿来作了一下！(14 千字)

*   **发信人**: `PowerBoy`
*   **时间**: `2003-11-03 17:48:16`

### 详细信息
****科学计算器 `1.8+`

*   **机器码**: `2654328163` --> `9E35D563`
*   **注册码**: `123456789`

软件的算法用到了 `CRC32` 的变形 `TABLE`（其中有很多元素不同），算法好像是作者在 `CRC32` 的基础上修改的。

不过，个人认为他的修改根本没有起到提高算法难度的作用，反倒把算法难度降低了很多（把算法变成了一个简单的查表算法）！！！不过和以前的版本相比已经有很大的进步了，希望作者多分析一下 `CRC32` 算法，真正理解这个算法之后再作修改！开始以为是 `CRC32` 算法等分析了发现....(不说废话了开始分析！)

输入注册码: `123456789`

怎么来到这里的我就不在多说了之前已经有人作了简单的分析了！

```assembly
:004AA0EF BE01000000      mov esi, 00000001  
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004AA12A(C)             //第 1 段计算(暂时叫 CRC32_1)  
|  
:004AA0F4 8D8558FFFFFF    lea eax, dword ptr [ebp+FFFFFF58]  //计算 1 开始  
:004AA0FA 8B15C4ED4A00    mov edx, dword ptr [004AEDC4]  
:004AA100 8A5432FF        mov dl, byte ptr [edx+esi-01]     //DL 开始按位取机器码  
:004AA104 E8ABA5F5FF      call 004046B4  
:004AA109 8B8558FFFFFF    mov eax, dword ptr [ebp+FFFFFF58]  
:004AA10F E8D4E9F5FF      call 00408AE8                      //TEMP:=STRTOINT(MN[A])  
:004AA114 8B048520CA4A00  mov eax, dword ptr [4*eax+004ACA20]//取 CRC32_TABLE[TEMP]  
:004AA11B 99              cdq                                 //用机器码的每位作为索引取 CRC32_TABLE  
:004AA11C 3345D8          xor eax, dword ptr [ebp-28]//EAX=EAX XOR SN(把 SN 作为初始值进行异或)  
:004AA11F 3355DC          xor edx, dword ptr [ebp-24]  
:004AA122 8945D8          mov dword ptr [ebp-28], eax  
:004AA125 8955DC          mov dword ptr [ebp-24], edx  
:004AA128 46              inc esi  
:004AA129 4B              dec ebx  
:004AA12A 75C8            jne 004AA0F4        //聪明的大家一定会想到上面的计算,CRC32_TABLE 只能用到前 10 个元素  
```

经过上面的计算: `123456789` --> `$09864527`

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004AA0ED(C)  
|  
:004AA12C 33F6            xor esi, esi  
:004AA12E B920CA4A00      mov ecx, 004ACA20  
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004AA15F(C)             //第 2 段计算(CRC32_2)  
|  
:004AA133 8B01            mov eax, dword ptr [ecx]       //EAX=EAX XOR CRC_TABLE[A]  
:004AA135 99              cdq  
:004AA136 3345D8          xor eax, dword ptr [ebp-28]  
:004AA139 3355DC          xor edx, dword ptr [ebp-24]  
:004AA13C 8945D8          mov dword ptr [ebp-28], eax  
:004AA13F 8955DC          mov dword ptr [ebp-24], edx  
:004AA142 8B8100020000    mov eax, dword ptr [ecx+00000200]//EAX=EAX XOR CRC_TABLE[128+A]  
:004AA148 99              cdq  
:004AA149 3345D8          xor eax, dword ptr [ebp-28]  
:004AA14C 3355DC          xor edx, dword ptr [ebp-24]  
:004AA14F 8945D8          mov dword ptr [ebp-28], eax  
:004AA152 8955DC          mov dword ptr [ebp-24], edx  
:004AA155 46              inc esi  
:004AA156 83C104          add ecx, 00000004  
:004AA159 81FE80000000    cmp esi, 00000080  
:004AA15F 75D2            jne 004AA133                 //这里是把 CRC32_TABLE 所有的元素进行异或  
                                            //所以就相当于和一个数异或这个值就是$DB3F761  
```

经过上面的计算 `$09864527` --> `$0435B246`

下面的数据就是内存中的表和标准的 `CRC32_TABLE` 有很多的不同可能是作者有意改变的！

```text
004ACA20  70 50 30 10 96 30 07 77  pP0?w0  
004ACA28  2C 61 0E EE BA 51 09 99  ,a 詈 Q.  
004ACB30  89 85 B1 71 1F B5 B6 06  墔眖刀 B1  
004ACB38  A5 E4 BF 9F 33 D4 B8 E8  ヤ繜 3 愿  
004ACA40  32 88 DC 0E A4 E8 DC 79  2 堒よ躽 DC  
004ACA48  1E E9 D5 E0 88 D9 D2 97  檎鄨僖  
....  
004ACD78  E1 77 B0 6F 77 47 B7 18  醱皁 wG? B0  
004ACD80  E6 5A 08 88 70 6A 0F FF  鎆坧 j08  
004ACD88  CA 3B 06 66 5C 0B 01 12  ?f\ 6  
004ACD90  FF 9E 65 8F 69 AE 63 F8  瀍廼甤  
```

`CRC_TABLE[0]=$10305070` 其实还有很多的变化的！

```assembly
:004AA161 DF6DD8          fild qword ptr [ebp-28]         //上面计算所得的值装入浮点寄存器  
:004AA164 83C4F4          add esp, FFFFFFFF4  
:004AA167 DB3C24          fstp tbyte ptr [esp]            //假计算,没有用途  
:004AA16A 9B              wait  
:004AA16B 8D8554FFFFFF    lea eax, dword ptr [ebp+FFFFFF54]  
:004AA171 E8E6FEF5FF      call 0040A05C  
:004AA176 8B9554FFFFFF    mov edx, dword ptr [ebp+FFFFFF54]//EDX=上面所得的数值  
:004AA17C B8C4ED4A00      mov eax, 004AEDC4  
:004AA181 E8A2A3F5FF      call 00404528  
:004AA186 BB08000000      mov ebx, 00000008  
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004AA1FB(C)             //计算 3(最后 2 位到最前面)  
|  
:004AA18B 8D45CC          lea eax, dword ptr [ebp-34]  
:004AA18E 50              push eax  
:004AA18F B901000000      mov ecx, 00000001  
:004AA194 8BD3            mov edx, ebx  
:004AA196 A1C4ED4A00      mov eax, dword ptr [004AEDC4]  
:004AA19B E844A8F5FF      call 004049E4  
:004AA1A0 8D45A8          lea eax, dword ptr [ebp-58]  
:004AA1A3 50              push eax  
:004AA1A4 8D7B02          lea edi, dword ptr [ebx+02]  
:004AA1A7 8BD7            mov edx, edi  
:004AA1A9 B901000000      mov ecx, 00000001  
:004AA1AE A1C4ED4A00      mov eax, dword ptr [004AEDC4]  
:004AA1B3 E82CA8F5FF      call 004049E4  
:004AA1B8 B8C4ED4A00      mov eax, 004AEDC4  
:004AA1BD B901000000      mov ecx, 00000001  
:004AA1C2 8BD3            mov edx, ebx  
:004AA1C4 E85BA8F5FF      call 00404A24  
:004AA1C9 BAC4ED4A00      mov edx, 004AEDC4  
:004AA1CE 8BCB            mov ecx, ebx  
:004AA1D0 8B45A8          mov eax, dword ptr [ebp-58]  
:004AA1D3 E894A8F5FF      call 00404A6C  
:004AA1D8 8BD7            mov edx, edi  
:004AA1DA B8C4ED4A00      mov eax, 004AEDC4  
:004AA1DF B901000000      mov ecx, 00000001  
:004AA1E4 E83BA8F5FF      call 00404A24  
:004AA1E9 8BCF            mov ecx, edi  
:004AA1EB BAC4ED4A00      mov edx, 004AEDC4  
:004AA1F0 8B45CC          mov eax, dword ptr [ebp-34]  
:004AA1F3 E874A8F5FF      call 00404A6C  
:004AA1F8 4B              dec ebx  
:004AA1F9 85DB            test ebx, ebx  
:004AA1FB 758E            jne 004AA18B   //循环移位(最后 2 位到最前面)  
```

`70627910` --> `10706279` (当然是把上面的数值先变成十进制然后在移位)
