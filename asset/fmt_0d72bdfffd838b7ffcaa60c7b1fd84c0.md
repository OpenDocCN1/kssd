
# 00427F80
```
00427F80     8B45 F8            mov eax,dword ptr ss:[ebp-8]
00427F83     3BF8             cmp edi,eax
//EAX=计算 CRC 的代码长度
00427F85     7C B6            jl short 00427F3D
00427F87     8B45 F0          mov eax,dword ptr ss:[ebp-10]
00427F8A     3B45 EC          cmp eax,dword ptr ss:[ebp-14]
//[ebp-14]中是计算出的校验值     比较校验值
00427F8D     74 05            je short 00427F94
//不跳就挂了
00427F8F     6A 03            push 3
00427F91     5F               pop edi
00427F92     EB 45            jmp short 00427FD9
```

## 五、修复校验

分析清楚了

`AutoIt` 取 `Overlay` 的最后 12 个字节异或 `AAAAAAAA` 得到校验值或者数据地址

1. 第一个校验数据 `XOR AAAAAAAA＝` 计算 CRC 的代码长度
2. 第二个校验数据 `XOR AAAAAAAA＝` 附加数据开始的地址
3. 第三个校验数据 `XOR AAAAAAAA＝` 文件校验值

看看我们用 `UPX` 对 `Test.AutoIt.CRC.exe` 脱壳得到的 `UnPacKed.exe` 文件信息：

1. 计算 CRC 的代码长度＝`00043B88`
2. 附加数据开始的地址＝`00043800`
3. 文件校验值＝需要跟踪－> `F928C0C3`

修复校验：

1. 第一个校验数据＝`00043B88 XOR AAAAAAAA＝AAAE9122`
2. 第二个校验数据＝`00043800 XOR AAAAAAAA＝AAAE92AA`

在 `UnPacKed.exe` 的校验数据开始的地址处修正这 2 个值，再看文件校验

载入 `UnPacKed.exe` 调试，`BP 00427F8A`，运行、中断，`[ebp-14]＝[0012ECA0]=F928C0C3`

3. 第三个校验数据＝`F928C0C3 XOR AAAAAAAA＝53826A69`

当然，第三个校验可以修改 `00427F8D jmp 00427F94` 直接去除，但是不推荐这样做

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
00043B80                    22 91 AE AA AA 92 AE AA
00043B90   69 6A 82 53
```

![](img/8f339b1517bc70143f2fd3cdd8c837b9.jpg)

修正这 12 个字节后，脱壳文件就可以正常运行了。

## 六、例子演示

论坛有个 `SW.exe` 也是 `AutoIt` 编译的文件

用 `UPX-Ripper` 脱壳得到 `SW.UnPacKed.exe`

载入 `SW.exe` 原版调试

```
00476760     60               pushad
//暂停在 UPX 外壳入口
00476761     BE 00E04400      mov esi,44E000
00476766     8DBE 0030FBFF    lea edi,dword ptr ds:[esi+FFFB3000]
0047676C     57               push edi
0047676D     83CD FF          or ebp,FFFFFFFF
00476770     EB 10            jmp short 00476782
004768CB     61               popad
004768CC     E9 17A9FCFF      jmp 004411E8
//F4 到这里
004411E8     55               push ebp
//OEP
004411E9     8BEC             mov ebp,esp
004411EB     6A FF            push -1
004411ED     68 28E44400      push 44E428
004411F2     68 50254400      push 442550
004411F7     64:A1 00000000   mov eax,dword ptr fs:[0]
004411FD     50               push eax
004411FE     64:8925 00000000 mov dword ptr fs:[0],esp
00441205     83EC 58          sub esp,58
00441208     53               push ebx
00441209     56               push esi
0044120A     57               push edi
0044120B     8965 E8          mov dword ptr ss:[ebp-18],esp
0044120E     FF15 A4D94600    call dword ptr ds:[46D9A4]     ; kernel32.GetVersion
```

正常的分析途径如上面所操作的。不过这里用个简便的方法来到检验值处理的地方

`Ctrl+F` 在整个段块搜索命令：`mov edi,0AAAAAAAA`

找到 `0043D50A` 处，下断，`F9` 运行后中断

```
0043D50A     BF AAAAAAAAA     mov edi,AAAAAAAA
0043D50F     317D FC          xor dword ptr ss:[ebp-4],edi
//[ebp-4]=[0012E844]=AAA81120  XOR AAAAAAAA=0002BB8A
//第一个校验值
//计算 CRC 的代码长度
//看看 0X0002BB8A 处数据，然后去 SW.UnPacKed.exe 里搜索，得到新的长度＝0005F58A
0043D512     6A 01            push 1
0043D514     6A 04            push 4
0043D516     53               push ebx
0043D517     E8 55230000      call 0043F871
0043D51C     FF36             push dword ptr ds:[esi]
0043D51E     313B             xor dword ptr ds:[ebx],edi
//[ebx]=[0012F864]=AAA80CAA XOR AAAAAAAA=0002A600
//第二个校验值
//附加数据开始的地址,UnPacKed.exe 里的附加数据开始地址＝0005E000
0043D520     8D45 F4          lea eax,dword ptr ss:[ebp-C]
0043D523     6A 01            push 1
0043D525     6A 04            push 4
0043D527     50               push eax
0043D528     E8 44230000      call 0043F871
0043D52D     317D F4          xor dword ptr ss:[ebp-C],edi
//[ebp-C]=[0012E83C]=5BC7EF2F XOR AAAAAAAA=F16D4585
//第三个校验值
//这个运算后的值是文件校验值
0043D530     33FF             xor edi,edi
0043D532     57               push edi
0043D533     57               push edi
0043D534     FF36             push dword ptr ds:[esi]
0043D536     E8 39220000      call 0043F774
0043D53B     8B45 FC          mov eax,dword ptr ss:[ebp-4]
0043D53E     83C4 48          add esp,48
0043D541     85C0             test eax,eax
0043D543     7E 4A            jle short 0043D58F
0043D545     8D8F 00000100    lea ecx,dword ptr ds:[edi+10000]
0043D54B     3BC8             cmp ecx,eax
0043D54D     7E 07            jle short 0043D556
0043D54F     2BC7             sub eax,edi
0043D551     8945 0C          mov dword ptr ss:[ebp+C],eax
0043D554     EB 07            jmp short 0043D55D
0043D556     C745 0C 00000100 mov dword ptr ss:[ebp+C],10000
0043D55D     FF36             push dword ptr ds:[esi]
0043D55F     037D 0C          add edi,dword ptr ss:[ebp+C]
0043D562     8D85 E8FEFEFF    lea eax,dword ptr ss:[ebp+FFFEFEE8]
0043D568     FF75 0C          push dword ptr ss:[ebp+C]
0043D56B     6A 01            push 1
0043D56D     50               push eax
0043D56E     E8 FE220000      call 0043F871
0043D573     83C4 10          add esp,10
0043D576     8D85 E8FEFEFF    lea eax,dword ptr ss:[ebp+FFFEFEE8]
0043D57C     8D4D F0          lea ecx,dword ptr ss:[ebp-10]
0043D57F     FF75 0C          push dword ptr ss:[ebp+C]
0043D582     50               push eax
0043D583     E8 25060000      call 0043DBAD
0043D588     8B45 FC          mov eax,dword ptr ss:[ebp-4]
0043D58B     3BF8             cmp edi,eax
0043D58D     7C B6            jl short 0043D545
0043D58F     8B45 F4          mov eax,dword ptr ss:[ebp-C]
0043D592     3B45 F0          cmp eax,dword ptr ss:[ebp-10]
//比较校验值
0043D595     74 05            je short 0043D59C
0043D597     6A 03            push 3
0043D599     5F               pop edi
0043D59A     EB 45            jmp short 0043D5E1
```

修复 `SW.UnPacKed.exe` 的校验：

1. 第一个校验数据＝`0005F58A XOR AAAAAAAA＝AAAF5F20`
2. 第二个校验数据＝`0005E000 XOR AAAAAAAA＝AAAF4AAA`

在 `SW.UnPacKed.exe` 的校验数据开始的地址处修正这 2 个值，再看文件校验

载入 `SW.UnPacKed.exe` 调试，`BP 0043D592`，运行、中断，`[ebp-10]＝[0012E838]=6C93F7B3`

3. 第三个校验数据＝`6C93F7B3 XOR AAAAAAAA＝C6395D19`

再修正最后一个校验值，可以运行了。

`Game Over`

```
             ,     \_/
           /| \_.-~/            \\\ ,        青春都一晌
          ( /~   /             \~-.\ |\\
          \`\\\\  /_/                  \\   ~\\ )          忍把浮名
      _-~~~-.)  )\__/;;,.              \\'  //'
  /'_,\\   --~   \\ ~~~-  ,;;\_\_\_(  (.-~~~-.        换了脱壳轻狂
 \`~ \_( ,_..--\\ (     ,;'' /    ~--   /.`\
  /~~//'   ' `~\\           ) /--..\_, ) \`~
  "  \`~"  "       `"       /~'\`\\    \`\\\\~~\
                   "     "   "~'  ""
             CracKed By :  fly
             2006-02-25 24:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
