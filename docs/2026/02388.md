
# 财智证券结算软件2.5 破解注册码分析！使用ollydbg 破解注册动画！高手莫入！ (1千字)

使用工具:W32Dasm9.0, trw 1.03! ollydbg 1.05

软件下载地址：http://nj.onlinedown.net/chaizistockwise.htm

运行软件，下`bpx 0050b67b`,我填入的注册码为 987654321，

点注册来到这儿：

```
:0050B67B E8E0F8FFFF              call 0050AF60

>>>>>>下 d eax看到假的注册码，

          好进去看看！

:0050B680 84C0

  test al, al

:0050B682 7454

  je 0050B6D8

:0050B684 A17CF05A00              mov eax,

dword ptr [005AF07C]

:0050B689 8B00

  mov eax, dword ptr [eax]

:0050B68B 8B4DFC

mov ecx, dword ptr [ebp-04]

\* Possible StringData Ref from Code Obj ->"RegisterCode"
```

跟进去我来到这儿：

```
:0050AF6A 8945FC

mov dword ptr [ebp-04], eax

:0050AF6D 8B45FC

mov eax, dword ptr [ebp-04]

:0050AF70 E8A394EFFF              call 00404418

          .
          .
          .

:0050AF99 E8F6FCFFFF              call 0050AC94
```

>>>>>>此处我下了`d eax`或`edx`就看到了真的

注册码：我的为8939－2343

机器码为：7209-3672

我于是使用ollydbg破解他的注册码，做了一个动画给大家看看！

大家看到我的注册码没有啊！我的注册码为：8939－2343

破解动画下载地址：http://person.longcity.net/home0/bcggroup/SoftWare/StockWise.rar
