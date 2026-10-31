
# see This 破解实战！ (5千字)

## 前言

最近正在W,]rg搞破解,但e的Cracker破得正g

不禁手痒,于是随便拉了个软件来试试:)

**软件下载:** `http://61.141.211.51/down/seedst.exe`

**软件简介：**

当你从网络上看到好东西的时候，你一定很想把整个画面传给朋友看。See This就是一个这麽好用的小工具，可以把整个屏幕“拍”下来，并且用Email传送出去。如果对方也是See
This的用户，画面就会立刻被传出去并且出现在他的屏幕上。要是对方没有使用See This，那麽这张图就会以jpg格式的Email夹文件传出去。

## 步骤

1、运行`trw2000`,再运行`seethis.exe`,选择输入`unlock`

随便填入:`abcdefghijk` 注:不能输入数字,嘿嘿,我 trace过,程序中有检测

2、下:`bpx hmemcpy`

按"确定"键,程序被拦,

下:`pmodule` 回到程序领空如下代码,开始trace:)

```
:004175F9 FF1540424300            Call dword ptr
[00434240]
:004175FF 8D7C240C                lea
edi, dword ptr [esp+0C]    <-d edi
:00417603 83C9FF
or ecx, FFFFFFFF
:00417606 33C0
  xor eax, eax
:00417608 33F6
  xor esi, esi
:0041760A F2
  repnz
:0041760B AE
  scasb
:0041760C F7D1
  not ecx
:0041760E 49
  dec ecx
:0041760F 7425
  je 00417636
:00417611 0FBE54340C              movsx edx,
byte ptr [esp+esi+0C]
:00417616 52
  push edx
:00417617 E834E50000              call 00425B50
:0041761C 88443410                mov
byte ptr [esp+esi+10], al
:00417620 83C404
add esp, 00000004
:00417623 8D7C240C                lea
edi, dword ptr [esp+0C]
:00417627 83C9FF
or ecx, FFFFFFFF
:0041762A 33C0
  xor eax, eax
:0041762C 46
  inc esi
:0041762D F2
  repnz
:0041762E AE
  scasb
:0041762F F7D1
  not ecx
:00417631 49
  dec ecx
:00417632 3BF1
  cmp esi, ecx
:00417634 72DB
  jb 00417611
```

以上一段代码,大家很容易看出是将输入的注册码,变换成大定字母.go on...!

```
:00417636 8D44240C                lea
eax, dword ptr [esp+0C]
:0041763A 6A08
  push 00000008
:0041763C 50
  push eax
:0041763D 68B09A4300              push 00439AB0
:00417642 E859DE0000              call 004254A0
:00417647 83C40C
add esp, 0000000C
:0041764A E821FEFFFF              call 00417470
<---注册比较核心,F8追入
:0041764F 85C0
  test eax, eax　　<---注册标志
:00417651 751A
  jne 0041766D  <--不为0则跳
:00417653 53
  push ebx
:00417654 E807010000              call 00417760
:00417659 83C404
add esp, 00000004
:0041765C B801000000              mov eax,
00000001
:00417661 5F
  pop edi
:00417662 5E
  pop esi
:00417663 5B
  pop ebx
:00417664 81C400010000            add esp, 00000100
:0041766A C21000
ret 0010
```

3、 在`call 00417470`处我们追入,看见如下代码:

```
:00417470 51
  push ecx
:00417471 53
  push ebx
:00417472 55
  push ebp
:00417473 8B2D60B64300            mov ebp, dword
ptr [0043B660]
:00417479 56
  push esi
:0041747A 57
  push edi
:0041747B 8B3D64B64300            mov edi, dword
ptr [0043B664]
:00417481 6A08
  push 00000008
:00417483 BE01000000              mov esi,
00000001
:00417488 68F09A4300              push 00439AF0
:0041748D 68B09A4300              push 00439AB0
:00417492 33DB
  xor ebx, ebx
:00417494 8974241C                mov
dword ptr [esp+1C], esi
:00417498 E8B3D50000              call 00424A50
:0041749D 83C40C
add esp, 0000000C
:004174A0 85C0
  test eax, eax
:004174A2 7479
  je 0041751D  <--不能跳哟,一跳则OVER!
:004174A4 6A08
  push 00000008
:004174A6 68E49A4300              push 00439AE4
:004174AB 68B09A4300              push 00439AB0
:004174B0 E89BD50000              call 00424A50
:004174B5 83C40C
add esp, 0000000C
:004174B8 85C0
  test eax, eax
:004174BA 7461
  je 0041751D  <--不能跳,跳则OVER!
:004174BC 33C0
  xor eax, eax
:004174BE 33C9
  xor ecx, ecx  <---|    ebx的由来:
:004174C0          mov cl, byte ptr [eax+00439AB0]
|  ABCEDFGH分别减去
:004174C6 83C1BF
add ecx, FFFFFFBF  |  41得到的
:004174C9 83F90F
cmp ecx, 0000000F  |
:004174CC 774F
  ja 0041751D        |
:004174CE 8BD6
  mov edx, esi      |
:004174D0 0FAFD1
imul edx, ecx      |
:004174D3 03DA
  add ebx, edx      |
:004174D5 C1E604
shl esi, 04        | 这里76543210
:004174D8 40
  inc eax            |
:004174D9 83F808
cmp eax, 00000008  |
:004174DC 7CE0
  jl 004174BE    --->|
:004174DE 3BEB
  cmp ebp, ebx  ebp中为153BAF0B
:004174E0 733B
  jnb 0041751D  <--ebp与ebx相等,就不会跳了!
      注册码不正确跳则GAMEOVER!
```

5、所以我们只要使`ebx`与`ebp`相等就可以了,

`ebx` 是输入注册码的变形来的,即`abcdefghijk`先换成大写字母`ABCDEDFHIJK`

再取前8位`ABCDEFGH`的ascII码,分别减去41,得到01234567.

`ebp` 中为`153BAF0B`,所以分别将1  5  3  B  A  F  0

B 加下41得到正确注册

码的ascII  码值为          42 46 44 4C 4B 50

41  4C

换算成为字母为  　　        B  F  D  L

K  P  A  L

正确的注册码为`LAPKLDFB`。

破解者： 十三少

China Cracking Group

2000.06.26
