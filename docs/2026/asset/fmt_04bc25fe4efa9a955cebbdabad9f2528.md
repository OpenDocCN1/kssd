
## 反汇编片段

```
004a86c0 0fb64608        movzx   eax,byte ptr [esi+8]

004a86c4 59              pop     ecx

004a86c5 5b              pop     ebx

004a86c6 5e              pop     esi

004a86c7 5f              pop     edi

004a86c8 5d              pop     ebp

004a86c9 c3              ret
```

从反汇编来看这个函数是尝试去设置一个图片到程序界面上显示，这个图片应该就是12306的图片验证码了。而它的再上层函数

```
0:008> !u 004a856f
Normal JIT generated code

z.a()

Begin 004a8550, size 67

004a8550 55              push    ebp
004a8551 8bec            mov     ebp,esp
004a8553 57              push    edi
004a8554 56              push    esi
004a8555 83ec10          sub     esp,10h
004a8558 33c0            xor     eax,eax
004a855a 8945e8          mov     dword ptr [ebp-18h],eax
004a855d 8945ec          mov     dword ptr [ebp-14h],eax
004a8560 8945f0          mov     dword ptr [ebp-10h],eax
004a8563 8945f4          mov     dword ptr [ebp-0Ch],eax
004a8566 8bf9            mov     edi,ecx
004a8568 8bcf            mov     ecx,edi
004a856a e8d140e9ff      call    0033c640 (z.i(), mdToken: 060000f1)
>>> 004a856f 85c0            test    eax,eax
004a8571 753d            jne     004a85b0
004a8573 8b0d80718f02    mov     ecx,dword ptr ds:[28F7180h] ("\?첓겙쭏摨瑿꾸\?\?")
004a8579 ba08000000      mov     edx,8
004a857e e86debffff      call    004a70f0 (<Module>.b(System.String, Int32), mdToken: 06000001)
004a8583 8bf0            mov     esi,eax
004a8585 8d4de8          lea     ecx,[ebp-18h]
004a8588 e883358f64      call    System_Drawing_ni+0x4bb10 (64d9bb10) (System.Drawing.Color.get_Black(), mdToken: 0600025f)
004a858d 8d45e8          lea     eax,[ebp-18h]
004a8590 ff700c          push    dword ptr [eax+0Ch]
004a8593 ff7008          push    dword ptr [eax+8]
004a8596 ff7004          push    dword ptr [eax+4]
004a8599 ff30            push    dword ptr [eax]
004a859b 8bd6            mov     edx,esi
004a859d 8bcf            mov     ecx,edi
004a859f e8ac40e9ff      call    0033c650 (z.a(System.String, System.Drawing.Color), mdToken: 060000fa)
004a85a4 b9e8030000      mov     ecx,3E8h
004a85a9 e82d848a62      call    mscorwks!ThreadNative::Sleep (62d509db)
004a85ae ebb8            jmp    004a8568
004a85b0 8d65f8          lea     esp,[ebp-8]
004a85b3 5e              pop     esi
004a85b4 5f              pop     edi
004a85b5 5d              pop     ebp
004a85b6 c3              ret
0:008> ? 3e8
Evaluate expression: 1000 = 000003e8
```

这个函数是在一个线程里面实现的，它会每隔1秒种就会调用上面两个函数，如果`004a856a`这里的函数返回非零的时候就跳出循环，否则就会一直去执行上面的两个函数。

## 崩溃流程总结

好了结合上面的信息，我们总结下这个崩溃的大致流程。该抢票软件在一运行起来之后就会创建一个线程，该线程会每隔1秒种 就会不断去尝试访问`e82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1P5h3&6S2L8h3W2U0i4K6u0W2x3e0t1K6x3o6k6Q4x3X3g2U0L8W2)9J5c8X3!0@1M7%4N6W2j5W2)9J5c8Y4m8S2M7%4y4o6L8$3c8W2b7h3y4@1K9h3!0F1i4K6u0W2k6r3!0Q4x3@1k6J5j5h3&6V1i4K6y4p5M7$3A6J5j5h3&6V1i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9K6b7W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4g2)9&6c8q4)9^5x3q4!0q4y4g2)9^5c8g2!0n7b7W2!0q4z5q4)9^5c8g2!0n7y4#2!0q4y4g2)9^5c8W2)9&6y4W2!0q4y4g2)9&6b7W2!0n7c8g2!0q4y4#2)9^5z5g2)9^5y4#2!0q4z5g2!0m8b7g2)9^5b7#2!0q4z5q4!0m8c8W2)9^5x3g2!0q4y4#2!0m8x3q4)9^5x3g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9^5y4q4!0n7y4W2!0q4y4g2)9&6x3q4)9^5c8g2!0q4y4W2)9^5b7g2)9^5b7g2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9&6b7W2!0n7c8g2!0q4y4#2)9^5z5g2)9^5y4#2!0q4y4W2)9&6z5q4!0n7c8g2!0q4y4#2!0m8y4q4!0n7b7g2!0q4y4g2)9^5z5q4!0n7x3q4!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4y4#2)9&6y4g2)9^5b7#2!0q4z5g2)9&6c8q4!0m8x3W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7c8q4)9^5y4W2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4g2!0n7c8g2)9^5z5q4!0q4y4q4!0n7z5q4)9^5c8q4!0q4y4g2!0n7z5g2!0n7z5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2)9&6y4#2!0n7y4W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4g2)9&6c8q4)9^5x3q4!0q4y4g2)9&6y4#2!0n7x3W2!0q4y4#2!0n7b7W2)9^5c8W2!0q4y4g2!0m8y4q4!0n7x3g2!0q4y4W2)9&6y4g2)9^5z5q4!0q4y4q4!0n7b7g2)9^5y4W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7c8q4)9&6b7#2!0q4z5q4)9^5x3q4)9^5y4g2!0q4y4q4!0n7z5g2)9&6c8W2!0q4y4W2!0n7x3W2!0m8x3g2!0q4y4W2)9&6b7#2)9^5z5g2!0q4y4g2!0m8y4q4)9^5y4q4!0q4y4#2)9&6x3q4)9^5y4W2!0q4y4g2!0m8y4g2!0n7c8q4!0q4y4q4!0n7z5g2)9^5b7W2!0q4y4g2)9&6x3q4)9^5c8g2!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4#2)9&6y4q4!0n7x3g2!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4z5q4)9^5y4#2!0m8b7g2!0q4z5q4!0n7b7g2!0m8b7W2!0q4y4W2)9^5b7g2)9&6b7W2!0q4y4g2)9^5y4#2!0n7b7g2!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4g2!0n7b7#2)9^5x3W2!0q4y4g2!0n7z5q4!0n7z5q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4g2!0n7y4q4!0m8z5g2!0q4y4W2!0n7b7g2)9^5x3#2!0q4y4q4!0n7b7g2)9^5y4W2!0q4x3#2)9^5x3q4)9^5x3R3`.

## 参考资料

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
