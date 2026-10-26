
### 汇编代码

```
004DB026   |.  50          push eax
004DB027   |.  8BC3        mov eax,ebx
004DB029   |.  33D2        xor edx,edx
004DB02B   |.  52          push edx
004DB02C   |.  50          push eax
004DB02D   |.  8BD7        mov edx,edi
004DB02F   |.  4A          dec edx
004DB030   |.  8BCB        mov ecx,ebx                        ;  减一16C5
004DB032   |.  8BC6        mov eax,esi
004DB034   |.  E8 CBFFFFFF call CDSnapsh.004DB004             ;  第一个CALL执行完毕
004DB039   |.  E8 0AA4F2FF call CDSnapsh.00405448             ;  \* Reference to: System.@\_llmod;用机器码/常数=X1
004DB03E   |.  E8 65A3F2FF call CDSnapsh.004053A8             ;  EAX回改变.@\_llmul;                X1\*机器码=X2
004DB043   |.  E8 00A4F2FF call CDSnapsh.00405448             ;  \* Reference to: System.@\_llmod;   X2/常数=X3
004DB048   |.  890424      mov dword ptr ss:[esp],eax
004DB04B   |.  895424 04   mov dword ptr ss:[esp+4],edx
004DB04F   |.  EB 11       jmp short CDSnapsh.004DB062
004DB051   |>  8BC6        mov eax,esi
004DB053   |.  33D2        xor edx,edx
004DB055   |.  F7F3        div ebx
004DB057   |.  8BC2        mov eax,edx
004DB059   |.  33D2        xor edx,edx
004DB05B   |.  890424      mov dword ptr ss:[esp],eax
004DB05E   |.  895424 04   mov dword ptr ss:[esp+4],edx
004DB062   |>  8B0424      mov eax,dword ptr ss:[esp]
004DB065   |.  8B5424 04   mov edx,dword ptr ss:[esp+4]
004DB069   |.  59          pop ecx
004DB06A   |.  5A          pop edx
004DB06B   |.  5F          pop edi
004DB06C   |.  5E          pop esi
004DB06D   |.  5B          pop ebx
004DB06E   \.  C3          retn  X3为返回值
```

-------------------------------------------------------------------

### 总结

系统首先回检测输入的长度是否是8位，

然后取假注册码的中间4位并转化为对应的整数，假设=`EDX`

在取机器码的最后5位并转化为对应的整数，假设=`EAX`

在取到常数`7C74B`，假设=`ECX`

接下来进行下面的运算：//返回值为`EAX`

`for(;EDX>=1;EDX--)`

   `{EAX=EAX/ECX;}`

   `EAX=EAX/ECX;//因为前面的判断条件是等于1`

`for(;EDX>=1;EDX--)//作者通过堆栈实现的。`

`{`

  `EAX=EAX%ECX;`

  `EAX=EAX*(机器码);`

  `EAX=EAX%ECX;`

`}`

接下来通过`GetVolumInforma`取到10位数，取后面的4位并转化为对应的整数，假设为`TEMP`

`if(EAX!=TEMP)`

   回将`EBX`清零

`else`

   将`EBX`低位置1

最后将`EBX`传给`EAX`，用来判断是否成功！！！

----------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
