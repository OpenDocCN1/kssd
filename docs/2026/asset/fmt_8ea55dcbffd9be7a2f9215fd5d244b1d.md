
解密后 `dump dll` `R10` 是基址，`R11` 是大小，如下图：

![](https://bbs.kanxue.com/upload/attach/201711/536985_k2cfmiv7gjc2fmx.jpg)

```
static main(void) 
    { 
        auto fp, begin, end, dexbyte; 
        fp = fopen("d:\\dump.dll", "wb"); //打开或创建一个文件 
        begin =  R10;              //dll基址 
        end = begin + R11;            //dll基址 + dll文件大小 
        for ( dexbyte = begin; dexbyte < end;dexbyte ++ ) 
        { 
            fputc(Byte(dexbyte), fp);     //按字节将其dump到本地文件中 
        }    
 }   
```

将 `dump` 出来的 `dll` 反编后如下图:

![](https://bbs.kanxue.com/upload/attach/201711/536985_ylgva7cqslevgml.jpg)

### 3. 在 `hook_mono_assembly_load_from_full` 主要是计算将解密 `dll` 的 `PE` 头清楚后的计算 `crc` 值。

## 七: `libengine` 模块分析

### 1. 在 `com/inca/security/Core/AppGuardEngine` 类中注册的第一个 `Native` 函数中通过 `dlopen`、`dlsym` 调用 `libengine.sox` 解密的 `so`

`JNI` 比函数 `.text:75C81168` `IlIIIIIIII` (文件偏移 `13168`) `(Landroid/content/Context;Ljava/lang/String;[B)I`

### 2. 通过 `dlopen`、`dlsym` 获取下面3个函数。

以下这几个函数被调用

`Initialize` (文件偏移 `27144`)

`CreateObject` (文件偏移 `2A2F4`)

`Clazz::IlIllllIIl` (文件偏移 `2B1B0`)

### 3. `Initialize` 函数

```
//反调试  
.text:7CEAE17A             loc_7CEAE17A                            ; CODE XREF: Initialize+32↑j 
.text:7CEAE17A CC 4B       LDR     R3, =(_ZN10lllllIIIll11IIlIlIIIlllE_ptr - 0x7CF87D44) 
.text:7CEAE17C EB 58       LDR     R3, [R5,R3] 
.text:7CEAE17E 1E 60       STR     R6, [R3]                        ; lllllIIIll::IIlIlIIIlll 
.text:7CEAE180 19 F0 B6 FD BL      Anit_Dbg                        ; 反调试(读进程状态) 
.text:7CEAE184 B8 42       CMP     R0, R7 
.text:7CEAE186 00 D0       BEQ     loc_7CEAE18A 
.text:7CEAE188 D4 E1       B       loc_7CEAE534 

//动态获取函数 
.text:7CEAE33A             loc_7CEAE33A                            ; CODE XREF: Initialize+1F2↑j 
.text:7CEAE33A 1A 60       STR     R2, [R3]                        ; Global::llIlIllllI 
.text:7CEAE33C 1E F0 DA FC BL      GetlibcFunc 
.text:7CEAE340 1F F0 76 FD BL      GetlibdvmFunc 
.text:7CEAE344 00 28       CMP     R0, #0 
.text:7CEAE346 04 D0       BEQ     loc_7CEAE352 
.text:7CEAE348 67 4B       LDR     R3, =(_ZN6Global11llllIIlIlIIE_ptr - 0x7CF87D44) 
.text:7CEAE34A EB 58       LDR     R3, [R5,R3] 
.text:7CEAE34C 04 93       STR     R3, [SP,#0x1A8+var_198] 
.text:7CEAE34E 01 23       MOVS    R3, #1 
.text:7CEAE350 1E E0       B       loc_7CEAE390   
```

### 4. `CreateObject` 函数

计算 `libengine.sox` 解密后的函数 `crc` (文件偏移 `2A1F6`)

```
.text:7CFA41F6             loc_7CFA41F6                            ; CODE XREF: IlllIIIlll::IlllIIIlll(_JNIEnv *,void *)+324↓j 
.text:7CFA41F6 39 4B       LDR     R3, =(_ZN6Global11lIIIIIIIIIIE_ptr - 0x7D07AD44) 
.text:7CFA41F8 07 9A       LDR     R2, [SP,#0xF8+var_DC] 
.text:7CFA41FA F3 58       LDR     R3, [R6,R3] 
.text:7CFA41FC 10 1C       MOVS    R0, R2                          ; void * 
.text:7CFA41FE 1A 60       STR     R2, [R3]                        ; Global::lIIIIIIIIII 
.text:7CFA4200 34 F0 26 F8 BL      GetCrc32                        ; 计算libengine.sox解密后的函数crc .text:7CFA4204 36 4B       LDR     R3, =(_ZN6Global11lIIlIlIIIlIE_ptr - 0x7D07AD44)  
```

计算 `CRC` 函数代码如下:
