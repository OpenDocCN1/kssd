
`DlgProc`

```
deflb DlgProc

push ebp

mov ebp,esp

mov eax,[ebp+0C]

cmp eax,0111

ofslb je command

cmp eax,010

ofslb je exit

xor eax,eax

pop ebp

ret 10
```

`Entry`

```
deflb Entry

sinvoke GetModuleHandleA/0

sinvoke DialogBoxParamW/eax/0FF/0/@DlgProc/0

cmp eax,1

ofslb je start

sinvoke ExitProcess/0
```

---

`F9` `Update`、`F11`、选择`entry`这个标签、然后`F3`复制标签地址、用`LordPE`打开、修改入口点为复制的值减去`1000000` `基地址`

运行、出现对话框、输入`1234567890`、`OK`、正确运行记事本、输入其他数字则出现非法操作

解释一下：

上面那段代码前面的空格不能省略、因为`deflb`会弹出一个红窗口、空格刚好可以把它关掉、另外`Hiew`里翻页时也必须多键入一个字符

这些代码要从下往上看、因为必须先定义标签再使用、所以要写出一段能直接`F12`的代码就只好倒着写了

--------------

`start:`

```
codexor 50,499602D2  //把0x50个字节和0x499602D2求xor (0x499602D2=1234567890,就是密码)
```

--------------

`cmdok:` `//解密`

```
sinvoke GetDlgItemInt/[ebp+8]/100/0/0

mov ebx,@start

mov ecx,14                  //  50/4=14

decrypt:

xor [ebx],eax

add ebx,4

ofslb loop decrypt

sinvoke EndDialog/[ebp+8]/1

or eax,1

pop ebp

ret 10
```

`exit:`

```
sinvoke EndDialog/[ebp+8]/0

or eax,1

pop ebp

ret 10
```

`command:`

```
mov eax,[ebp+10]

and eax,0FFFF

cmp eax,101         //处理OK按钮

ofslb je cmdok

cmp eax,102         //处理cancel按钮

ofslb je exit

xor eax,eax

pop ebp

ret 10
```

`DlgProc:` `//对话框过程`

```
push ebp

mov ebp,esp

mov eax,[ebp+0C]

cmp eax,0111         //WM\_COMMAND

ofslb je command

cmp eax,010          //WM\_CLOSE

ofslb je exit

xor eax,eax

pop ebp

ret 10
```

`Entry:` `//新入口点`

```
sinvoke GetModuleHandleA/0

sinvoke DialogBoxParamW/eax/0FF/0/@DlgProc/0  //打开对话框

cmp eax,1

ofslb je start //解密后跳到原来的入口点

sinvoke ExitProcess/0  //Cancel就结束进程
```

这里只要按了`OK`、不管密码对不对都解密、而错误的密码会导致解密后的代码出现非法操作

********************************************************************************

终于写玩了

欢迎继续修改这个代码、修改后也请给我一份 `Email:DQuixote@qq.com`

最后感谢你耐心地看到这里
