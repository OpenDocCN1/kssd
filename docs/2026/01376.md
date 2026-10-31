
# [分享]植物大战僵尸 <秒杀>

看见有人分享关于植物大战僵尸的贴，自己也打开`CE`玩了玩，顺利在无尽模式中玩到40关以上

内容见下

## 思路

扫描未知数值

当普通僵尸受到攻击时搜索减少的值

重复`N`次，获得地址

对地址下写入断点分析其代码来到下图处

```
点击此处，编辑代码
```

![](https://bbs.kanxue.com/upload/attach/201801/744806_xsmxkxdd6e4rc7v.jpg)

可以看出`EDI`里是实时的血量，这里将其改为`xor edi,edi`即可直接将血量归0

接着运行游戏，发现有铁桶僵尸和拿报纸的僵尸不受上述代码约束，向上翻看代码发现`call`由下图来

![](https://bbs.kanxue.com/upload/attach/201801/744806_gd3jyyg2v0myedi.jpg)

仔细辨别代码，发现两处`cmp`有重大嫌疑，将`JE`改为`JNE`时发现已实现僵尸秒杀。时间仓促，将`CE`代码附上以便大家玩耍

此代码将铁桶类僵尸视为普通僵尸

```
[ENABLE]

//code from here to '[DISABLE]' will be used to enable the cheat

alloc(newmem,2048)

label(returnhere)

label(originalcode)

label(exit)



newmem: //this is allocated memory, you have read,write,execute access

//place your code here



originalcode:

//je PlantsVsZombies.exe+13186B

jne PlantsVsZombies.exe+13186B

push esi

mov eax,ebp



exit:

jmp returnhere



"PlantsVsZombies.exe"+131861:

jmp newmem

returnhere:

 

[DISABLE]

//code from here till the end of the code will be used to disable the cheat

dealloc(newmem)

"PlantsVsZombies.exe"+131861:

je PlantsVsZombies.exe+13186B

push esi

mov eax,ebp

//Alt: db 75 08 56 8B C5
```

此代码将报纸僵尸类视为普通僵尸

```
[ENABLE]

//code from here to '[DISABLE]' will be used to enable the cheat

alloc(newmem,2048)

label(returnhere)

label(originalcode)

label(exit)



newmem: //this is allocated memory, you have read,write,execute access

//place your code here



originalcode:

//je PlantsVsZombies.exe+131856

jne PlantsVsZombies.exe+131856

test bl,01



exit:

jmp returnhere



"PlantsVsZombies.exe"+13183F:

jmp newmem

returnhere:



[DISABLE]

//code from here till the end of the code will be used to disable the cheat

dealloc(newmem)

"PlantsVsZombies.exe"+13183F:

je PlantsVsZombies.exe+131856

test bl,01

//Alt: db 75 15 F6 C3 01
```

此代码将血量归零

```
[ENABLE]

//code from here to '[DISABLE]' will be used to enable the cheat

alloc(newmem,2048)

label(returnhere)

label(originalcode)

label(exit)



newmem: //this is allocated memory, you have read,write,execute access

//place your code here



originalcode:

//sub edi,[esp+20]

xor edi,edi

mov [esp+1C],eax



exit:

jmp returnhere



"PlantsVsZombies.exe"+13130F:

jmp newmem

nop

nop

nop

returnhere:



[DISABLE]

//code from here till the end of the code will be used to disable the cheat

dealloc(newmem)

"PlantsVsZombies.exe"+13130F:

sub edi,[esp+20]

mov [esp+1C],eax

//Alt: db 2B 7C 24 20 89 44 24 1C
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
