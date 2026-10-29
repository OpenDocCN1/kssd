
```
//R4寄存器存放的是InterpState结构
LOAD:0003FF5C                 LDRB            R1, [R4,#4]   ; 取出key，InterpState->key
LOAD:0003FF5E                 MOVS            R6, #0x31 ; '1'
LOAD:0003FF60                 PUSH            {R1}
LOAD:0003FF62                 POP             {R2}
LOAD:0003FF64                 ANDS            R2, R6
LOAD:0003FF66                 LDR             R0, loc_402C8
LOAD:0003FF68                 PUSH            {R0}
LOAD:0003FF6A                 POP             {R3}
LOAD:0003FF6C                 BICS            R3, R1
LOAD:0003FF6E                 ORRS            R3, R2
LOAD:0003FF70                 MOVS            R2, #0xEF00
LOAD:0003FF74                 LSLS            R1, R1, #8
LOAD:0003FF76                 ANDS            R2, R1
LOAD:0003FF78                 BICS            R0, R1
LOAD:0003FF7A                 ORRS            R0, R2
LOAD:0003FF7C                 EORS            R0, R3
LOAD:0003FF7E                 LDR             R1, [R4]    ; 取出pc， Interpstate->pc
LOAD:0003FF80                 LDRH            R4, [R1]    ; 取出指令

//执行完invoke-super后
LOAD:000463EC loc_463EC
LOAD:000463EC
LOAD:000463EC                 LDR             R0, [R4]
LOAD:000463EE                 ADDS            R0, #6
LOAD:000463F0                 STR             R0, [R4]  ; InterpState->pc = InterpState->pc + 6
LOAD:000463F2                 BL              loc_3FF5C ; 跳转到解释器开头，执行下一条指令。
```

### 数字壳`invoke-super`指令分析

`invoke-super`指令包含了函数调用的过程，可以看到dalvik解释器虚拟寄存器栈是比较复杂的，设计很多数据结构。目前为止数字壳的相关结构中，并未发现类似的结构。所以想分析下数字壳的解释器是如何处理函数调用的。调试分析后发现，数字壳的解释器其实并未实现真正的函数调用，它是通过调用`jni`中的`CallVoidMethod`方法来实现函数调用。

处理`invoke-super`的handler为`sub_4878C`，流程如下：

1. 提取`invoke-super`指令中包含的`MethodID`，从dex中获取到`DexMethodId`结构，获取函数名。
2. 从`DexMethodId`中获取`ClassId`。
3. 利用获取的`ClassId`可以获取到类名，利用`jni->FindClass`获取类。
4. 利用`jni->GetMethod`获取函数。
5. 准备函数参数。
6. 利用`jni->CallVoidMehtod`调用函数，实现`invoke-method`。

```
int  sub_4878C(int a1, bool methodCallRange, JNIEnv *a3, DexInfo *a4, StackInfo *a5, InterpState *a6, int opNumber, void *a8)
{
  //节选
  a1a = v8;
  dex_base = *v8;
  dex_header = *(a1a + 16);
  key = *(a6 + 4);                              // 取出key
  v11 = (key << 8) | key;
  DexMethodId = dex_base + *(dex_header + 92) + 8 * (*(*a6 + 2) ^ v11);
  v13 = dex_base + *(dex_header + 60);
  methodName = (dex_base + *(v13 + 4 * *(DexMethodId + 4)));   //获取函数名
  do
    v15 = *methodName++ < 0;
  while ( v15 );
  DexMethodId2 = (dex_base + *(dex_header + 92) + 8 * (*(*a6 + 2) ^ ((key << 8) | key)));
  proto = (dex_base
         + *(v13
           + 4 * *(dex_base + *(dex_header + 68) + 4 * *(dex_base + *(dex_header + 76) + 12 * *(DexMethodId + 2) + 4))));
  do
    v17 = *proto++;                        //获取函数简单声明
  while ( v17 < 0 );
  v18 = *(*a6 + 4);
  if ( v52 == 1 )
    thisReg = v18 ^ ((key << 8) | key);     //this参数
  else
    thisReg = (v18 ^ key) & 0xF;
  v20 = 0;
  if ( v51 )
  {
    v21 = *(*(a5 + 20) + 4 * thisReg);
    if ( !v21 || (v20 = *v21) == 0 )
    {
      v30 = ((*v53)->FindClass)(v53, "java/lang/NullPointerException");
      .....
    }
  }
  arg0 = v20;                                   // this object
  v62 = v53;
  v63 = 0;
  if ( ((*v53)->ExceptionCheck)(v53) )
    v63 = 0;
  if ( v51 != 2 && v51 != 4 )
  {
    sub_610BC(a1a, v53, *DexMethodId2);         // 获取到classid所指定的类
    return sub_48AE2(v32, v33, v34, v35, a5);   // 利用jni->CallVoidMethod调用父类方法
  }

//48EAA: 构建invoke-super除this外的参数
//49770：调用jni->CallVoidMethod方法
```

数字壳比较复杂，通过分析可以学到很多东西，比如各种反调试，linker，适配很多版本的动态加载，解释器等，感谢数字公司提供的免费加固。笔记如果有错误，欢迎指正，也希望大佬们可以交流下其他vmp的实现思路。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
