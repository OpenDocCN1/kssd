
好了，到此你已经没事了，唯一需要的就是修复导入表入口和 EP 了。

```
0101039F
          50
PUSH EAX
 010103A0            0385 22040000
ADD EAX,[DWORD SS:EBP+422]<====修改OEP的RVA程VA
 010103A6
      59            POP ECX
010103A7            0BC9
  OR ECX,ECX
 010103A9            8985
A8030000  MOV [DWORD SS:EBP+3A8],EAX<====+写入
 010103AF
        61            POPAD
+
 010103B0            75 08
    JNZ SHORT notepad.010103BA    +
 010103B2
        B8 01000000    MOV EAX,1
                +
 010103B7
          C2 0C00        RETN 0C
                      +
 010103BA            68 00000000
PUSH 0=========================+
 010103BF
  C3            RETN<==========================返回原始程序
```

---

Enjoy it:)

DiKeN/iPB

---

我相信，看了这篇文章，你应该会了 ASPack 的脱壳了。

关于完全修复，我就不做赘述，精通 PE 结构的人可以修复，新手没有必要修复了。

---

### 结束语

标准 ASPack 的壳，就这样简单。都是这样，要还原成原样也没问题。

tELock 的壳，也使用了 aPLib 作为其压缩引擎，不过它有一次加密/解密。

UPX 也使用了 aPLib 这个压缩引擎。

aPLib 引擎，以前的版本没有了。可以到`http://apack.cjb.net`或者`http://home19.inet.tele.dk/jibz/apack/`或者到 iPB 论坛，工具栏下载。有兴趣的可以看一下。
