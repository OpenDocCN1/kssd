

# Enigma protector V1.02.build.4.00 主程序脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-777.htm`](https://www.pediy.com/kssd/pediy07/pediy7-777.htm)

Enigma protector V1.02.build.4.00 主程序脱壳  
  
原贴以及下载以及汉化文件 http://www.unpack.cn/viewthread.php?tid=2210&fpage=1  
或下载:http://www.enigma.xost.net/   
或 http://www.pediy.com/tools/PACK/Protectors/enigma/enigma1.02.zip   
  
2k sp4  
flyodbg  
importrec  
winhex  
  
  
简单介绍一下我的脱壳过程，没有仔细看，某些地方说不清楚  
  
  
一， 解码后 dump  
清除所有断点，忽略所有异常  
对 401000 下内存写入断点(1 个字节就可以了)  
等啊等  
比较慢  
狂异常  
顺便 WC 一下，回来后发现断在这里  
  00FCC1E9          8807             mov byte ptr ds:\[edi\],al  // 断在这里  
  00FCC1EB          47               inc edi  
  00FCC1EC          00D2             add dl,dl  
  00FCC1EE          75 05            jnz short 00FCC1F5  
  
清除内存断点，往下拉，找 retn  
  
  00FCC304          5D               pop ebp  
  00FCC305          2B7D 40          sub edi,dword ptr ss:\[ebp+40\]  
  00FCC308          897C24 1C        mov dword ptr ss:\[esp+1C\],edi  
  00FCC30C          61               popad  
  00FCC30D          5D               pop ebp  
  00FCC30E          C3               retn  
  00FCC30F          894424 1C        mov dword ptr ss:\[esp+1C\],eax  
  00FCC313          61               popad                   // 你要在这里下断也行，不过很快发现出口是下面一个  
  00FCC314          8945 FC          mov dword ptr ss:\[ebp-4\],eax  
  00FCC317          8B45 FC          mov eax,dword ptr ss:\[ebp-4\]  
  00FCC31A          59               pop ecx  
  00FCC31B          5D               pop ebp  
  00FCC31C          C2 0800          retn 8                  // 这里下断  
  
上面返回  
  00FCC15A          8945 FC          mov dword ptr ss:\[ebp-4\],eax   // 00FCC31C 返回到这里  
  00FCC15D          F6C3 04          test bl,4  
  00FCC160          74 0A            je short 00FCC16C  
  00FCC162          8B45 FC          mov eax,dword ptr ss:\[ebp-4\]  
  00FCC165          50               push eax  
  00FCC166          57               push edi  
  00FCC167          E8 08000000      call 00FCC174  
  00FCC16C          5F               pop edi  
  00FCC16D          5E               pop esi  
  00FCC16E          5B               pop ebx  
  00FCC16F          59               pop ecx  
  00FCC170          5D               pop ebp  
  00FCC171          C2 1400          retn 14                        // F4 到这里  
  
然后  
  00FCC112          8B53 05          mov edx,dword ptr ds:\[ebx+5\]   // 00FCC171 返回到这里  
  00FCC115          8BC6             mov eax,esi  
  00FCC117          E8 6C65FEFF      call 00FB2688  
  00FCC11C          83C3 35          add ebx,35  
  00FCC11F          837B 0D 00       cmp dword ptr ds:\[ebx+D\],0  
  00FCC123        ^ 75 AE            jnz short 00FCC0D3             //这个 jnz 如果跳上去的话，继续去判断下面的段是否要解码  
  00FCC125          5F               pop edi  
  00FCC126          5E               pop esi  
  00FCC127          5B               pop ebx  
  00FCC128          C3               retn                           // F4 到这里  
  
这时 F4 到 00FCC128 后，发现第一个区段解码了  
  00401000  04 10 40 00 03 07 42 6F 6F 6C 65 61 6E 01 00 00  @.Boolean..  // 显然解码了  
  
  
这时可以用 OD dump 了，此时不 dump， 后面就被重定位了  
  
  
二，继续前进  
解码后 alt-m 对 401000 段下内存访问断点，F9 运行  
  
  00FB283D          F3:A5            rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  // 断在这里  
  00FB283F          89C1             mov ecx,eax  
  00FB2841          83E1 03          and ecx,3  
  00FB2844          83C6 03          add esi,3  
  00FB2847          83C7 03          add edi,3  
  00FB284A          F3:A4            rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
  00FB284C          FC               cld  
  00FB284D          5F               pop edi  
  00FB284E          5E               pop esi  
  00FB284F          C3               retn                               //取消内存断点， F4 下来  
  
这一段不知道干嘛的，它将解码后的代码段 copy 其它地方去，不去管它  
返回后  
  00FCB9FA          B8 40000000      mov eax,40  
  00FCB9FF          2B43 39          sub eax,dword ptr ds:\[ebx+39\]  
  00FCBA02          2BF0             sub esi,eax  
  00FCBA04          03F8             add edi,eax  
  00FCBA06          8BC3             mov eax,ebx  
  00FCBA08          E8 B3F5FFFF      call 00FCAFC0  
  00FCBA0D          EB 12            jmp short 00FCBA21  
  00FCBA0F          8D5403 4D        lea edx,dword ptr ds:\[ebx+eax+4D\]  
  00FCBA13          8BC7             mov eax,edi  
  00FCBA15          8BCE             mov ecx,esi  
  00FCBA17          E8 F46DFEFF      call 00FB2810  
  00FCBA1C          0173 39          add dword ptr ds:\[ebx+39\],esi  
  00FCBA1F          33F6             xor esi,esi  
  00FCBA21          85F6             test esi,esi  
  00FCBA23        ^ 77 B4            ja short 00FCB9D9                 // 这里会跳上去，下面 F4 就是要跳过这里  
  00FCBA25          5F               pop edi  
  00FCBA26          5E               pop esi  
  00FCBA27          5B               pop ebx  
  00FCBA28          C3               retn                              // F4 下来  
  
  
  
继续 alt-m 对 401000 段下内存访问断点， F9 运行   
  
  00FCE45A          890F             mov dword ptr ds:\[edi\],ecx        // 断在这里  
  00FCE45C          40               inc eax  
  00FCE45D          4A               dec edx  
  00FCE45E        ^ 75 D0            jnz short 00FCE430  
  00FCE460          43               inc ebx  
  00FCE461          FF0C24           dec dword ptr ss:\[esp\]  
  00FCE464        ^ 75 B2            jnz short 00FCE418  
  00FCE466          5A               pop edx  
  00FCE467          5D               pop ebp  
  00FCE468          5F               pop edi  
  00FCE469          5E               pop esi  
  00FCE46A          5B               pop ebx  
  00FCE46B          C3               retn  
  
00FCE45A   mov dword ptr ds:\[edi\],ecx  对代码段中的 jmp \[IAT\] 的 IAT 地址重定位了，重定位到高地址，并且 IAT 里面加密了。  
当前改写的第一个是  
  00401428   FF25 90316000    jmp dword ptr ds:\[603190\]  
00FCE45A   mov dword ptr ds:\[edi\],ecx 这句话过后  
改成了  00401428   FF25 A1170601    jmp dword ptr ds:\[10617A1\]  
10617A1 中的内容是  
  01062818          55               push ebp  
  01062819          8BEC             mov ebp,esp  
  0106281B          6A FF            push -1  
  0106281D          68 C813F877      push 77F813C8  
  01062822          68 647EFB77      push 77FB7E64  
  01062827          64:A1 00000000   mov eax,dword ptr fs:\[0\]  
  0106282D          50               push eax  
  0106282E          64:8925 00000000 mov dword ptr fs:\[0\],esp  
  01062835          51               push ecx  
  01062836          51               push ecx  
  01062837          83EC 10          sub esp,10  
  0106283A          53               push ebx  
  0106283B          56               push esi  
  0106283C          57               push edi  
  0106283D          8B5D 08          mov ebx,dword ptr ss:\[ebp+8\]  
  01062840          8B43 10          mov eax,dword ptr ds:\[ebx+10\]  
  01062843          85C0             test eax,eax  
  01062845          68 DFF0F877      push 77F8F0DF  
  0106284A          C3               retn  
显然是一个加密的 API  
  
  
三，记下 10617A1 和 01062818 这 2 个地方，重来  
清除所有断点，忽略所有异常  
完成上面的第一部分(因为第一部分完成后 01062818 的空间被分配了)  
对 01062818 下内存写入断点(1 个字节够了) F9 跑  
中间大概会断几次无关的地方，放过，当断在这里时  
  00FB282A          F3:A4            rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\] //断在这里是需要的  
  00FB282C    5F               pop edi  
  00FB282D          5E               pop esi  
  00FB282E          C3               retn                                  //清除内存断点 F4 到这里  
  
上面 F4 到那儿返回后还是个 retn，再 F8 返回后是这里  
  00FCE728          01FE             add esi,edi                           //返回到这里  
  00FCE72A          01FD             add ebp,edi  
  00FCE72C        ^ E9 56FFFFFF      jmp 00FCE687  
  00FCE731          807C24 10 00     cmp byte ptr ss:\[esp+10\],0  
  00FCE736          75 16            jnz short 00FCE74E  
  00FCE738          C606 68          mov byte ptr ds:\[esi\],68  
  00FCE73B          8BC6             mov eax,esi  
  00FCE73D          8BD0             mov edx,eax  
  00FCE73F          42               inc edx  
  00FCE740          892A             mov dword ptr ds:\[edx\],ebp  
  00FCE742          83C0 05          add eax,5  
  00FCE745          C700 C3000000    mov dword ptr ds:\[eax\],0C3  
  00FCE74B          83C6 06          add esi,6  
  00FCE74E          8B0424           mov eax,dword ptr ss:\[esp\]  
  00FCE751          8D04C0           lea eax,dword ptr ds:\[eax+eax\*8\]  
  00FCE754          8B15 B8CFFD00    mov edx,dword ptr ds:\[FDCFB8\]  
  00FCE75A          8B4C24 08        mov ecx,dword ptr ss:\[esp+8\]  
  00FCE75E          894C02 01        mov dword ptr ds:\[edx+eax+1\],ecx     //这一行是对 10617A1 的改写  
  00FCE762          FF0424           inc dword ptr ss:\[esp\]  
  00FCE765          FF4C24 14        dec dword ptr ss:\[esp+14\]  
  00FCE769        ^ 0F85 D5FEFFFF    jnz 00FCE644                         //跳上去处理下一个加密的输入表  
  00FCE76F          83C4 18          add esp,18  
  00FCE772          5D               pop ebp  
  00FCE773          5F               pop edi  
  00FCE774          5E               pop esi  
  00FCE775          5B               pop ebx  
  00FCE776          C3               retn  
  
00FCE75E    mov dword ptr ds:\[edx+eax+1\],ecx  把这一行 nop 掉(3 个 nop) 然后 F4 到下面的 retn  
再把这 3 个 nop 恢复(撤消选择)，我也不知道有没有自校验，反正这样不会错  
这样一来，像 10617A1 这样所有的被重定位的 IAT 中几乎都是真实的 API 地址了  
  
  
alt-m 对 401000 段下内存访问断点  
和第二部分中的一样，又到这里了  
  00FCE45A          890F             mov dword ptr ds:\[edi\],ecx        //这一次到这里时\[ecx\]中的是真实 API 了  
  00FCE45C          40               inc eax  
  00FCE45D          4A               dec edx  
  00FCE45E        ^ 75 D0            jnz short 00FCE430  
  00FCE460          43               inc ebx  
  00FCE461          FF0C24           dec dword ptr ss:\[esp\]  
  00FCE464        ^ 75 B2            jnz short 00FCE418  
  00FCE466          5A               pop edx  
  00FCE467          5D               pop ebp  
  00FCE468          5F               pop edi  
  00FCE469          5E               pop esi  
  00FCE46A          5B               pop ebx  
  00FCE46B          C3               retn  
  
我们只要将\[ecx\]写入\[\[edi\]\]中，输入表就巧妙回来了  
清除内存断点  
找一块空地  
修改如下：  
  00FCE45A         /E9 A1890100      jmp 00FE6E00  
  00FCE45F         |90               nop  
  00FCE460         |43               inc ebx  
  
  00FE6E00          50               push eax                        //空地  
  00FE6E01          53               push ebx  
  00FE6E02          8B01             mov eax,dword ptr ds:\[ecx\]  
  00FE6E04          8B1F             mov ebx,dword ptr ds:\[edi\]  
  00FE6E06          8903             mov dword ptr ds:\[ebx\],eax  
  00FE6E08          5B               pop ebx  
  00FE6E09          58               pop eax  
  00FE6E0A          40               inc eax  
  00FE6E0B          4A               dec edx  
  00FE6E0C        ^ E9 4F76FEFF      jmp 00FCE460  
  
然后 F4 到上面的 00FCE46B   retn 就全都处理完了，安全起见把上面的恢复回来  
  
  
四， 找到数据段(DATA 段)  
  
继续 alt-m 对 401000 段下内存访问断点  
  00FD6B6D          0108             add dword ptr ds:\[eax\],ecx        // 断在这里  
  00FD6B6F          EB 35            jmp short 00FD6BA6  
  00FD6B71          8B0D B492FD00    mov ecx,dword ptr ds:\[FD92B4\]  
  00FD6B77          8B1D 24D3FD00    mov ebx,dword ptr ds:\[FDD324\]  
  00FD6B7D          2B99 DC000000    sub ebx,dword ptr ds:\[ecx+DC\]  
  00FD6B83          8B15 B492FD00    mov edx,dword ptr ds:\[FD92B4\]  
  00FD6B89          8B92 E9000000    mov edx,dword ptr ds:\[edx+E9\]  
  00FD6B8F          2BDA             sub ebx,edx  
  00FD6B91          25 FFFFFF7F      and eax,7FFFFFFF  
  00FD6B96          0305 24D3FD00    add eax,dword ptr ds:\[FDD324\]  
  00FD6B9C          8B0D B492FD00    mov ecx,dword ptr ds:\[FD92B4\]  
  00FD6BA2          2BC2             sub eax,edx  
  00FD6BA4          0118             add dword ptr ds:\[eax\],ebx  
  00FD6BA6          8305 28D3FD00 04 add dword ptr ds:\[FDD328\],4  
  00FD6BAD          A1 28D3FD00      mov eax,dword ptr ds:\[FDD328\]  
  00FD6BB2          8B00             mov eax,dword ptr ds:\[eax\]  
  00FD6BB4          85C0             test eax,eax  
  00FD6BB6        ^ 75 88            jnz short 00FD6B40               // 跳上去继续处理  
  00FD6BB8          33C0             xor eax,eax                      // 清除内存断点， F4 到这里  
  00FD6BBA          5A               pop edx  
  
清除内存断点，这个 add dword ptr ds:\[eax\],ecx 是将数据段中的数据重定位到高地址，走过这句后  
发现完整的数据段(还没有被重定位)在  
010F0000  00 00 00 00 00 00 00 00 00 00 00 00 02 8D 40 00  ............岪.  
010F0010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
010F0020  00 00 00 00 32 13 8B C0 02 00 8B C0 00 8D 40 00  ....2 嬂.嬂.岪.  
010F0030  00 8D 40 00 00 8D 40 00 00 00 00 00 00 00 00 00  .岪..岪.........  
010F0040  A8 21 40 00 38 23 40 00 B8 26 40 00 00 CB CC C8  ?@.8#@.?@..颂  
010F0050  C9 D7 CF C8 CD CE DB D8 DA D9 CA DC DD DE DF E0  勺先臀圬谫受蒉哙  
010F0060  E1 E3 00 E4 E5 8D 40 00 45 72 72 6F 72 00 8B C0  徙.溴岪.Error.嬂  
010F0070  52 75 6E 74 69 6D 65 20 65 72 72 6F 72 20 20 20  Runtime error  
010F0080  20 20 61 74 20 30 30 30 30 30 30 30 30 00 8B C0    at 00000000.嬂  
010F0090  30 31 32 33 34 35 36 37 38 39 41 42 43 44 45 46  0123456789ABCDEF  
  
于是乎右键->备份->保存到数据文件，保存下来， 一会儿贴回去  
  
  
五，飞向光明之颠  
该做的都做完了  
alt-m 对 401000 段下内存访问断点, F9 运行   
  
  004F9918          E8 5BCAF0FF      call Enigma.00406378             //断在这里  
  004F991D          8B1D E4300F01    mov ebx,dword ptr ds:\[10F30E4\]   //你看数据段的被重定位了，不过我们前面 dump 了一份  
  004F9923          8B03             mov eax,dword ptr ds:\[ebx\]  
  004F9925          E8 E648F9FF      call Enigma.0048E210  
  004F992A          8B0D 782E0F01    mov ecx,dword ptr ds:\[10F2E78\]  
  004F9930          8B03             mov eax,dword ptr ds:\[ebx\]  
  004F9932          8B15 D0524E00    mov edx,dword ptr ds:\[4E52D0\]  
  
断在这里后看到堆栈中的值是 A7FC4C79 和 EBX 一样， EAX 中的值是 004F949C  
所以前面被偷掉的 oep 完整的应该是  
  
  004F990C          55               push ebp  
  004F990D          8BEC             mov ebp,esp  
  004F990F          83C4 F0          add esp,-10  
  004F9912          53               push ebx  
  004F9913          B8 9C944F00      mov eax,Enigma.004F949C  
  
  
  
六，整合  
  
到了这里，可以打开 ImportIAT 修输入表了，发现有 7，8 个还是加密了  
这 7,8 个并不难， ctrl-g 过去看一下就能知道是什么了  
比如这个  
  00FCDC9C          68 B3DCFC00      push 0FCDCB3  
  00FCDCA1          64:FF35 00000000 push dword ptr fs:\[0\]  
  00FCDCA8          64:8925 00000000 mov dword ptr fs:\[0\],esp  
  00FCDCAF          31C0             xor eax,eax  
  00FCDCB1          3100             xor dword ptr ds:\[eax\],eax  
  00FCDCB3          8B4424 0C        mov eax,dword ptr ss:\[esp+C\]  
  00FCDCB7          C780 B8000000 D0>mov dword ptr ds:\[eax+B8\],0FCDCD0  
  00FCDCC1          8B0D 98CFFD00    mov ecx,dword ptr ds:\[FDCF98\]  
  00FCDCC7          8988 B0000000    mov dword ptr ds:\[eax+B0\],ecx  
  00FCDCCD          31C0             xor eax,eax  
  00FCDCCF          C3               retn  
  00FCDCD0          64:8F05 00000000 pop dword ptr fs:\[0\]  
  00FCDCD7          83C4 04          add esp,4  
  00FCDCDA          C3               retn  
费劲搞了个异常，其实就是 mov eax, \[FDCF98\]这句话  
到\[FDCF98\]里面看看，就知道这个 API 是 GetCommandLineA  
  
  
前面 dump 出来的 PE + dump 出来的数据段 + 输入表修复 + oep 改成 F990C 就全都 OK 了  
如果你想把壳段删了，你还要多做一步就是把 tls 段的四个值贴到原来的地方 

 4dd466d0e1e0cc123afbe504da2462ca