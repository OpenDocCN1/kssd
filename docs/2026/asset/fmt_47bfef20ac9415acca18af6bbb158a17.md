
# 对某驱动的逆向（src written by Kmdkit4D）

这应该算是鄙人在看雪的第一帖了，由于鄙人菜鸟无知，可能会有很多错误，还望各位高手多多指教。

首先废话下，感谢`DonQuixote`前辈的教导（现在很少看到他了，可能是学业问题？bless），他让我明白了许多的东西，无论是技术还是非技术方面的，其次，感谢`MickeyLan`前辈，`MickeyLan`开发的`KmdKit4D`让我这种Delphi爱好者也可以在驱动的世界里慢慢摸索，还要感谢女王，没有女王当初的启发，我也不可能开始学习驱动。最后感谢CSDN Delphi版的众版主们，虽然他们可能一生也不会和驱动打交道，但他们给了我极大的支持。

最后膜拜下“快雪时晴”这位HUST的学长（反正我喊谁都是学长，至少现在如此，RT……）兼大牛（众：你有完没完？）

进入正题，某同学的U盘中毒，某个叫做`Ressdt.sys`的文件入住，虽然看名字都知道是干什么的，但是为了自娱自乐，还是拿出来放到IDA里面玩了下。

个人比较喜欢上来就看`idata`段，可以对整个文件有个比较大致的理解和认识，可以看到有标准的`stdcall`，包括`ProbeForWrite`，`RtlInitUnicodeString`等，这些都是驱动的重要因素，例如添加符号链接，虽然不是必须的，但这样做可以使用户模式的代码更容易找到驱动，RT。翻了下，在`IoCreateDevice`的`stdcall`里面找到了`KeServiceDescriptorTable`这个东西，大致都知道是如何工作了吧？直接指向`KeServiceDescriptorTable`，进行SSDT的还原。

然后看`text`段，开头的两句汇编证明了这种猜测：

```
mov     eax, offset KeServiceDescriptorTable
mov     eax, [eax]
retn
```

下面还有一个`Dispatch`例程的建立：

```
push    ebp
mov     ebp, esp
push    ebx
mov     ebx, [ebp+arg_4]
push    offset Format   ; "DisPatchCreate!"
call    DbgPrint
```

`ProbeForWrite`的定义在`9d2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3#2K6y4K6V1%4x3e0l9^5i4K6u0W2j5i4y4H3P5q4!0q4y4g2!0m8y4q4)9^5y4q4!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4g2)9^5z5q4!0n7x3q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2!0m8c8W2)9&6y4q4!0q4z5q4!0n7c8g2)9^5x3#2!0q4y4W2)9&6b7#2)9^5z5g2!0q4z5q4!0n7y4W2!0m8x3#2!0q4y4q4!0n7z5g2)9&6c8W2!0q4y4W2!0m8c8W2)9&6y4q4!0q4z5q4!0n7c8g2)9^5x3#2!0q4y4g2!0n7b7#2!0n7b7g2!0q4y4g2!0m8y4q4!0m8y4#2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2)9&6z5q4!0m8c8W2!0q4c8W2!0n7b7#2)9^5b7@1W2p5b7g2!0q4y4#2)9&6b7W2!0n7y4q4!0q4y4W2)9^5c8g2!0m8y4g2!0q4y4W2!0m8x3q4)9^5y4#2!0q4y4#2!0m8y4q4!0n7b7g2!0q4y4q4!0n7b7g2)9^5y4W2m8J5L8$3u0W2c8X3!0J5g2%4u0A6N6r3g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5k6Q4z5o6u0Q4c8e0k6Q4z5e0g2Q4b7U0m8Q4c8e0g2Q4z5p5g2Q4z5p5u0Q4c8e0k6Q4b7e0m8Q4z5o6S2Q4c8e0W2Q4b7e0q4Q4b7V1q4Q4c8e0g2Q4b7V1q4Q4z5p5k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0g2Q4z5p5k6Q4z5p5q4Q4c8e0g2Q4z5e0m8Q4z5o6c8Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0g2Q4z5p5k6Q4z5o6u0Q4c8e0k6Q4z5e0g2Q4b7U0m8Q4c8f1k6Q4b7V1y4Q4z5f1p5`.
```
push    4               ; Alignment
push    4               ; Length
mov     eax, [ebp+Address]
push    eax             ; Address
call    ProbeForWrite
```

下面是一段极其醒目的代码，玩过SSDT的都知道是什么了：

```
add     esp, 0Ch
cli //IO指令，关闭中断，那个啥啥，邪恶的东西开始了
mov     eax, cr0 //修改CR0寄存器，修改内核必备良药
and     eax, 0FFFEFFFFh
mov     cr0, eax
mov     eax, [edi]//通过ProbeForRead获得一个索引，应该是DeviceIoControl的方法，如有错误欢迎指正
mov     edx, ebx
add     edx, edx
add     edx, edx
add     edx, [ebp+var_10]
mov     [edx], eax
mov     eax, cr0//再改回来，不可写属性
or      eax, 10000h
mov     cr0, eax
sti//开中断
xor     ebx, ebx
jmp     short loc_103E0
```

Have you ever read the 《sky book》?开个玩笑……忘记说了，DDK好像也没声明`KeServiceDescriptorTable`，需要自己手动添加，恩恩。

之后有一些杂乱的代码，比如利用Short Jump实现`IoCompleteRequest`的完成请求……这些不再赘述，毕竟不是特别的重点，比如`IoCreateSymbolicLink`创建符号链接，都是用函数，转换起来也很简单，就不多说了。

源代码由`dcc32`和`KmdKit4D`一同编译。

最后插播一则广告：欢迎访问`214K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3E0E0k6r3E0A6N6o6c8V1i4K6u0W2L8X3g2@1i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2B7M7%4m8Q4c8e0u0Q4z5o6m8Q4b7e0k6Q4c8e0u0Q4z5o6m8Q4b7e0j5`.
