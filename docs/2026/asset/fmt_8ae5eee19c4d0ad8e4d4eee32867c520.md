
```asm
10007179  mov edi,ebp
1000717B  rep movs dword ptr es:[edi],dword ptr d>
1000717D  pop edi
1000717E  pop esi
1000717F  pop ebp
10007180  add esp,18
10007183  retn
10007184  cmp ax,word ptr ss:[ebp+8]              ; 与第5组注册码比较
10007188  jnz short sss.100071B8
1000718A  mov dx,word ptr ss:[esp+12]
1000718F  cmp dx,word ptr ss:[ebp+6]              ; 与第4组注册码比较
10007193  jnz short sss.100071B8
10007195  mov ax,word ptr ss:[esp+10]
1000719A  cmp ax,word ptr ss:[ebp+4]              ; 与第3组注册码比较
1000719E  jnz short sss.100071B8
100071A0  mov cx,word ptr ss:[esp+E]
100071A5  cmp cx,word ptr ss:[ebp+2]              ; 与第2组注册码比较
100071A9  jnz short sss.100071B8
100071AB  pop edi
100071AC  mov word ptr ss:[ebp+A],572
100071B2  pop esi
100071B3  pop ebp
100071B4  add esp,18
100071B7  retn
```

---

计算第3.4.5组主册码的`call`

```asm
10007050  mov ecx,dword ptr ss:[esp+8]
10007054  mov eax,dword ptr ss:[esp+4]            ; call前面保存的一个值
10007058  and eax,0FFFF
1000705D  mov edx,dword ptr ds:[ecx+14]           ; call前面保存的一个值
10007060  add eax,edx
10007062  mov edx,dword ptr ds:[ecx+C]            ; 里面的值是0x50AEF62F
10007065  add edx,434713EA
1000706B  imul eax,edx
1000706E  xor edx,edx
10007070  mov dx,word ptr ds:[ecx+10]             ; 里面的值是0x15F8
10007074  lea eax,dword ptr ds:[eax+edx+D431]     ; eax=eax+edx+0xd431
1000707B  mov dword ptr ds:[ecx+14],eax           ; 保存temp
1000707E  shr eax,10
10007081  retn                                    ; ax的值就是注册码
```

---

## 算法总结：

由输入第一组注册码，依次得到真正第2，3，4，5组册码与输入比较，全符合则能够正确注册。

1. 第2组生成过程：`eax=code1*0x2063`取`ax`低8位`a`和高8位`b`组成新数`ab`，`ab+code1=code2`
2. 第3组生成过程：取`code1`前2位`a`，`code2`后2位`b`，组成`ba`，`eax=(ba+(code1+code2))*(0x50AEF62F+0x434713EA)`；`code3=(eax+0x15F8+0xD431)`高16位
3. 第4组生成过程：`eax=(code3^(code1+code2+code3)+temp)*(0x50AEF62F*0x434713EA)`（这里`temp=code2<<16+code3`）；`code3=(eax+0x15F8+0xD431)`高16位
4. 第5组生成过程：`eax=(code4^(code1+code2+code3+code4)+temp)*(0x50AEF62F*0x434713EA)`（这里`temp`由3产生）；`code3=(eax+0x15F8+0xD431)`高16位

写了一个简单算法注册机，只是把作者的简单改写下，没有什么技术含量，见笑

```c
void main(){

        unsigned short code[5]={0x1234,0,0,0,0};

    unsigned short * p=code;

        time_t tt;

    srand((unsigned) time(&tt));

        code[0]=rand()%0xffff;

        __asm{

                xor ecx,ecx

                mov ebx,p

                mov cx,word ptr [ebx]

                mov eax,ecx

                imul eax,02063h

                mov dl,ah

                mov dh,al

                add edx,ecx

                mov word ptr [ebx+2],dx ;第2组注册码

                ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

                shl dx,8

                mov dl,ch

                mov ax,word ptr [ebx+2]

                and eax,0ffffh

                add eax,ecx

                and edx,0ffffh

                add eax,edx

                imul eax,093F60A19h

                add  eax,0ea29h

                shr eax,010h

                mov word ptr [ebx+4],ax ;第3组注册码

                ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

                xor ecx,ecx

                mov edx,eax

                add dx,word ptr [ebx+2]

                add dx,word ptr [ebx]

                xor eax,edx

                mov cx,word ptr [ebx+2]

                shl ecx,010h

                mov cx,word ptr [ebx+4]

                add eax,ecx

                imul eax,093F60A19h

                add  eax,0ea29h

                mov ecx,eax

                shr eax,010h

                mov word ptr [ebx+6],ax ;第4组注册码

                ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

                add edx,eax

                sub edx,01e92h

                xor eax,edx

                and eax,0ffffh

                add eax,ecx

                imul eax,093F60A19h

                add  eax,0ea29h

                shr eax,010h

                mov word ptr [ebx+8],ax ;第5组注册码

        }

        printf("注册码：");

        for(int i=0;i<5;i++)

        printf("%X ",code[i]);

        printf("\n");

}
```

注册信息保存在`HKEY_LOCAL_MACHINE\SOFTWARE\Image2Lcd`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
