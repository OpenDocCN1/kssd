
## 解密与代码块处理

```
0040880B             ; -----------------------------------------------------------
0040880B 83 EF 04    sub edi, 4
0040880E E9 12 FE FF FF    jmp near ptr byte_408625
00408813             ; -----------------------------------------------------------
00408813             _done_:            ; bpm here to wait till it finishes
                                        下 BPM 等待直到完成
```

这一段很难懂，因为代码 `FLOW`（流程）不是一条直线，我希望通过色彩的标记使大家能了解清楚一些。算法与先前摘录的代码算法类似，只不过代码块长度计算不同，而且 `TABLE ENTRIES` 在解密方面未于利用。基本上 `TABLE ENTRIES` 对于加密代码段的解密工作是通过 `40859A` 段的 `CRC` 循环完成的。如果你在 `4085A6` 上下断点，就会发现 `EAX` 就是解密的地址。我有两个解决 `CRC` 校验的办法，一种是把整个原始代码块复制到另一区域，然后修改 `CRC` 程序，在新的区域完成校验；还有一种就是再次利用 `SOFTICE` 断点。我赞成后一种，因为它简单易行。

一般的，我会让 `PELOCK LOADER` 继续对每个 `TABLE ENTRIES` 执行解密，然后在加密代码块的前后插入适当的调用。但是，要想在 `4085A6` 处设断并把解密的地址存入表（`TABLE`）中，我们至少得在结束时拥有一张解密表。

```
bpm 408813 x      ；整个 INSERTION 循环结束时的地址断点
bpm 4085a6 x do "dd esi+4 eax" ；把 EAX 中解密值存回表中
                                  ESI 指针还原（-4->+4）
```

当 `SOFTICE` 到达最终的断点时，你会在数据窗口中发现新的包含加密代码块地址的解密表。剩下的工作就是通过插入一些代码解密出最后的表中所有的代码块。

```
A EIP
mov ecx, 4A; 表大小
load_lodsd; 从表中读取 1ENTRY
push dword ptr [eax+5];保存
pushad
call eax; 虚拟原始 EXE 执行程序
popad
pop dword ptr [eax+5]; 复原代码块属性值
mov word ptr [eax], 7EB;修改（PATCH）Call Decryptor JUMP OVER（跳过）
mov ebx, word ptr [eax+5] ; 读取
ror ebx, 3; 得到代码块长度
add eax, ebx
add eax, 19; 计算 Call Encryptor 地址
mov word ptr [eax], 7EB; 替换（REPLACE）Call Encryptor JUMP OVER（跳过）
dec ecx; 表大小减 1
test ecx, ecx
jne load_; 如果未结束则获取下一个 TABLE ENTRY
jmp eip; 大循环, 在此 DUMP
```

接下来，在修改完成另两处 `INSERTION` 程序后，工作做完。因此当 `PELOCK` 重新运行并进入大循环后，`EXE` 中所有的加密代码块便完全被解密，并且 `DECRYPTORS` 和 `RE-ENCRYPTORS` 调用也通过修改跳过。

## 5. 找到 OEP

既然我们已经正确 `DUMP` 程序并且解密、修正了所有的代码，接下来的工作就是运行 `IMPREC` 并对 `IMPORT DIRECTORY` 进行纠正了。我们的目的当然是找到 `OEP`。看一下 `DISASSEMBLY` 和 `HEX` 显示，我们发现第一节（`FIRST SECTION`）以大量字符串开始，代码紧随其后，所以猜想 `OEP` 一定在字符串的尾部。

另一类一般做法是 `PELOCK` 从哪里跳转到 `OEP`。在前述的三对 `INSERTION` 程序完成执行后，答案很快就揭晓了。

```
00407578 E8 2B 66 00 00      call insert_crypt_engine_3 ; insertion routine call
00407578             ; -----------------------------------------------------------
0040757D 90 90 90 90 90 90+  db 28h dup(90h); that is responsible for inserting
004075A5             ; -----------------------------------------------------------
004075A5 E8 0C 73 00 00      call insert_crypt_engine_2 ; those Call Decryptor/Encryptor
004075A5             ; -----------------------------------------------------------
004075AA 90 90 90 90 90 90+  db 28h dup(90h)   ; 进入 EXE 主代码段 004075D2             ; -----------------------------------------------------------
004075D2 E8 C5 07 00 00      call unknown_import_stuff_and_insert_crypt_engine_1
004075D2             ;
;-----------------------------------------------------------
004075D7 90 90 90 90 90 90+  db 28h dup(90h); 代码仍然被加密
<-------------省略-------------->; 在到达前最后解密
00407AA1 5D               pop ebp; OEP.
00407AA1             ;
;-----------------------------------------------------------
00407AA2 90 90 90 90 90 90+  db 124h dup(90h)
00407BC6             ; -----------------------------------------------------------
00407BC6 68 1E 1D 40 00      push 401D1Eh    ; OEP!!!
00407BC6             ; -----------------------------------------------------------
00407BCB 90 90 90 90 90 90+  db 23h dup(90h)
00407BEE             ; -----------------------------------------------------------
00407BEE C3               retn
```

因此，当找到任何 `INSERTION ROUTINE` 后，你可以检查堆栈从而发现从哪里调用，然后下 `BPM`。你应该 `DUMP LOADER` 并且看看 `IDA`。尽管在到达 `OEP` 之前，会有一些循环和 `SHE` 等给你的进程带来麻烦，但是 `IDA` 会帮你解决这一切并最终找到 `OEP`。

## 6. 总结

（省略了一些废话：-））

总的来说，`PELOCK` 是一个很好的壳，破解的过程也充满刺激。假使 `PELOCK` 在加壳（`PACKING`）时就已经插入（`INSERTING`）了 `DECRYPTOR` 和 `ENCRYPTOR` 的调用，并且删去了包含解密代码块地址的表的话，我想我们一定很难或者不能确定解密出所有的代码块。现在有一个想法，那就是 `LOADER` 经常会在不同的偏移上装载，因此在加壳时 `PELOCK` 就会发现调用的距离使用修改地址 `CALL DWORD PTR [XXXXXXX]` 来代替地址 `XXXXXXX`。

我希望你能喜欢我的解说。欢迎探讨，如果有什么意见建议，请在 `FRAVIA MESSAGE BOARD` 上留言。

感谢所有关心帮助我的朋友！

最后编辑：2003/1/30

干杯！

`CRUSADER`
`4dd466d0e1e0cc123afbe504da2462ca`
