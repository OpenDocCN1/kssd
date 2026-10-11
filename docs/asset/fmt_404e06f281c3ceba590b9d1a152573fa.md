
# ASM 学习笔记之简单循环与判断

> 原文：[`www.pediy.com/kssd/pediy12/137594.html`](https://www.pediy.com/kssd/pediy12/137594.html)

20110722 学习简单汇编指令 `for` `if` 等

生成一段 `for` 代码反编译看之

## 1. `for` 循环分析

```
6: int myfunction(int a,int b)
7: {
01381380  push        ebp    /
01381381  mov         ebp,esp
01381383  sub         esp,0D8h
01381389  push        ebx
0138138A  push        esi
0138138B  push        edi
0138138C  lea         edi,[ebp-0D8h]
01381392  mov         ecx,36h
01381397  mov         eax,0CCCCCCCCh
0138139C  rep stos    dword ptr es:[edi]      到这里位置是标准的函数入口把参数从右到左分别压入堆栈 没什么好学习的 重点看下面
```

```
8:  int c=a+b;
0138139E  mov         eax,dword ptr [a]  //首先把 a 变了赋值给 eax
013813A1  add         eax,dword ptr [b]  //同上不过换成了 B 变量
013813A4  mov         dword ptr [c],eax  ///三句指令获得的就是 int c=a+b；
```

```
9:  int i;
10:  for(i=0;i<50;i++)
013813A7  mov         dword ptr [i],0  //初始化 I=0
013813AE  jmp         myfunction+39h (13813B9h) 然后直接跳到循环的开始处
```

```
013813B0  mov         eax,dword ptr [i] ///这是从下面的无条件跳转 到这里进行循环的++操作
013813B3  add         eax,1 //上面把 I 赋值到 eax 这里把 eax 加 1
013813B6  mov         dword ptr [i],eax //然后把 eax 再赋值给 I 下面接着进行对比
```

```
013813B9  cmp         dword ptr [i],32h  //i 与 50 对比 这里的 50=32（16 进制）转换为 8 进制
013813BD  jge         myfunction+4Ah (13813CAh)  //jge 跳转条件满足则跳转 这里的条件就是只 i 不小于 50
```

```
11:  {
12:    c= c+i;
013813BF  mov         eax,dword ptr [c]   //如果上述跳转实现不了的话就执行这一句初始化 C 的值
013813C2  add         eax,dword ptr [i]  // eax+i 就等于 C 的值
013813C5  mov         dword ptr [c],eax   //然后把 eax 赋值给 C 就获得了 C=C+I 之所以这么麻烦因为是 debug 编译模式下 具体为啥不得而知
13:  }
```

```
013813C8  jmp         myfunction+30h (13813B0h) //跳转回去进行循环
14:  return c;
013813CA  mov         eax,dword ptr [c]  //在汇编语句中 所有的变量值都是保存在 eax 中
15:
16: }
```

`Push` 操作会把栈顶 -4 就是 `esp-4` 而 `pop` 相应的 +4

通过以上代码我们得出结论 `for` 的 `asm` 结构就是

- `mov` <循环变量的值> <初始值> 也就是上面给 `I` 赋值的地方
- `Jmp B` 也就是赋值后进入第一次循环对比的地方
- `A`：（改动循环变量值）
- `B`：`cmp`（循环变量）（最大值）
  - `Jge` 跳出循环 大于等于则跳
  - （主循环体）.....
- `Jmp A` 如果 `jge` 不能形成这跳到 `A` 处

## 2. `do` 循环学习

```
9:  do{
10:    c=c+i;
001E341E  mov         eax,dword ptr [c]
001E3421  add         eax,dword ptr [i]       这三句代码的意思不用说已经很明白了
001E3424  mov         dword ptr [c],eax
```

```
11:  }while(c<100);
001E3427  cmp         dword ptr [c],64h 循环对比的地方 64h 这种 16 进制已经讲了转换为 8 进制
001E342B  jl          myfunction+1Eh (1E341Eh) //跳回去 jl 小于则跳
```

```
12: return c;
001E342D  mov         eax,dword ptr [c] //返回 C 的值
13: }
```

因为 `DO` 没有去修改变量值所以代码灰常简单与河蟹

## 3. `while` 循环

接下来是 `while` 循环 此循环体在 `C++` 编写中经常用到

### 反汇编代码

```
9: while(c<100)
013E341E  cmp         dword ptr [c],64h  直接对比
013E3422  jge         myfunction+2Fh (13E342Fh)  大于等于则跳
```

```
10: {
11:   c = c+i;
013E3424  mov         eax,dword ptr [c]
013E3427  add         eax,dword ptr [i]
013E342A  mov         dword ptr [c],eax 到这里就是把运算结果赋值给 C
12: }
```

```
013E342D  jmp         myfunction+1Eh (13E341Eh) 跳入循环体
13:
14: return c;
013E342F  mov         eax,dword ptr [c] 返回最后值
```

获得结构如下

- `A Cmp`（循环变量值）（最大值）
- `Jge B` 大于等于则跳
- ......
- `Jmp A`
- `B` 循环外

`- --------------------------循环体大致就是 cmp jge jg jmp 等 下面是 if 等语法`

## 4. `if` 判断

```
10: if(c>0&&c<10)
01012E3E  cmp         dword ptr [c],0    //我们可以看到 首先 c>0 对比
01012E42  jle         myfunction+43h (1012E63h) //如果小于的话就跳走
01012E44  cmp         dword ptr [c],0Ah //如果上一个条件满足 则判断 c 是否小于 10
01012E48  jge         myfunction+43h (1012E63h) 大于等于则跳
```

```
11: {
12:   printf("c>0");
01012E4A  mov         esi,esp
01012E4C  push        offset string "c>0" (101573Ch)
01012E51  call        dword pt _printf (10182B4h)]                   这里是输出部分先不管
01012E57  add         esp,4
01012E5A  cmp         esi,esp
01012E5C  call        @ILT+455(__RTC_CheckEsp) (10111CCh)
01012E61  jmp         myfunction+7Fh (1012E9Fh) 这里的话直接调走了 因为 if 条件已经全部满足则不需要进行下面的判断
13:
14: }
```

```
15: else if(c>10&&c<100)
01012E63  cmp         dword ptr [c],0Ah
01012E67  jle         myfunction+68h (1012E88h)
01012E69  cmp         dword ptr [c],64h
01012E6D  jge         myfunction+68h (1012E88h)
```

```
16: {
17:   printf("c>10&&c<100");
01012E6F  mov         esi,esp
01012E71  push        offset string "c>10&&c<100" (10157A0h)
01012E76  call        dword ptr [__imp__printf (10182B4h)]
01012E7C  add         esp,4
01012E7F  cmp         esi,esp
01012E81  call        @ILT+455(__RTC_CheckEsp) (10111CCh)
18: }
```

```
19: else
01012E86  jmp         myfunction+7Fh (1012E9Fh)
20: {
21:   printf("c>10&&c<100");
01012E88  mov         esi,esp
01012E8A  push        offset string "c>10&&c<100" (10157A0h)
01012E8F  call        dword ptr [__imp__printf (10182B4h)]
01012E95  add         esp,4
01012E98  cmp         esi,esp
01012E9A  call        @ILT+455(__RTC_CheckEsp) (10111CCh)
22: }
23:
24: return c;
01012E9F  mov         eax,dword ptr [c]
25:
}
```

大体上我们获得 `if` 判断就是 `cmp` 与 `jle` 条件不满足直接调走 如满足则到下一个条件分支

## 5. `switch-case` 分支判断

```
001C2E3E  mov         eax,dword ptr [c]
001C2E41  mov         dword ptr [ebp-0C4h],eax                             首先把 C 移动到 ebp-0c4h 的地方
001C2E47  cmp         dword ptr [ebp-0C4h],0 与 0 进行对
001C2E4E  je          myfunction+3Bh (1C2E5Bh) 大于则跳
001C2E50  cmp         dword ptr [ebp-0C4h],1 与 1 进行对比
001C2E57  je          myfunction+52h (1C2E72h) 大于则跳
001C2E59  jmp         myfunction+6Bh (1C2E8Bh)
```

```
11: {
12: case 0:
13:   printf("C>0");
001C2E5B  mov         esi,esp
001C2E5D  push        offset string "c>0" (1C573Ch)
001C2E62  call        dword ptr [__imp__printf (1C82B4h)]
001C2E68  add         esp,4
001C2E6B  cmp         esi,esp
001C2E6D  call        @ILT+455(__RTC_CheckEsp) (1C11CCh)
14:
```

```
15: case 1:
16:   {
17:     printf("C>10&&c<100");
001C2E72  mov         esi,esp
001C2E74  push        offset string "C>10&&c<100" (1C57A0h)
001C2E79  call        dword ptr [__imp__printf (1C82B4h)]
001C2E7F  add         esp,4
001C2E82  cmp         esi,esp
001C2E84  call        @ILT+455(__RTC_CheckEsp) (1C11CCh)
18:     break;
001C2E89  jmp         myfunction+82h (1C2EA2h)
19:   }
```

```
20: default:
21:   printf("C>10&&c<100");
001C2E8B  mov         esi,esp
001C2E8D  push        offset string "C>10&&c<100" (1C57A0h)
001C2E92  call        dword ptr [__imp__printf (1C82B4h)]
001C2E98  add         esp,4
001C2E9B  cmp         esi,esp
001C2E9D  call        @ILT+455(__RTC_CheckEsp) (1C11CCh)
22: }
23: return c;
```

以上为判断与循环语句的 `asm` 分析
