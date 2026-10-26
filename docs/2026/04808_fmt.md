
# flashget0.96去除广告条方法 (3千字)

软件名称：网际快车FlashGet(JetCar) 国际版

整理日期：2001.6.14

最新版本：0.96

文件大小：1401KB

软件授权：免费软件

使用平台：Win9x/Me/NT/2000

URL：`http://sd.onlinedown.net/down/fgf096.zip`

安装后先运行一下，把语言设为english，发现`help`菜单中有`Remove Banner...`项。

运行`w32dasm`，找字符串`remove banner`，没找到。

再找字符串`remove`，找到这一段。

```
* Reference To: USER32.DeleteMenu, Ord:0087h      <==(2) 哈，用的是deletemenu，你想到了什么？
            |
:0044C2BC 8B3D20064B00            mov edi, dword
ptr [004B0620]
:0044C2C2 6A00
  push 00000000
* Possible Reference to String Resource ID=32846: "Register to remove the banner"
      <==(1) w32dasm会找到这里
            |
:0044C2C4 684E800000              push 0000804E
:0044C2C9 50
  push eax
:0044C2CA FFD7
  call edi
:0044C2CC 85C0
  test eax, eax
:0044C2CE 740D
  je 0044C2DD
:0044C2D0 8B4E04
mov ecx, dword ptr [esi+04]
:0044C2D3 6800040000              push 00000400
:0044C2D8 6A01
  push 00000001
:0044C2DA 51
  push ecx
:0044C2DB FFD7
  call edi        <==(3)这句话去掉菜单中的remove banner...项
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0044C28F(C), :0044C2B7(C), :0044C2CE(C)
<==(4)程序中有3处比较，可能会跳过(3) call edi
          用trw设断点，会发现程序会由44c28f直接过来,
          现在goto code location 44c28f
|
:0044C2DD 8B4C241C                mov
ecx, dword ptr [esp+1C]
:0044C2E1 5F
  pop edi
:0044C2E2 5E
  pop esi
:0044C2E3 5D
  pop ebp
:0044C2E4 5B
  pop ebx
:0044C2E5 64890D00000000          mov dword ptr fs:[00000000],
ecx
:0044C2EC 83C418
add esp, 00000018
:0044C2EF C3
  ret
----------------------------------------------------------------------------------------------
:0044C288 E893A6FCFF              call 00416920
:0044C28D 85C0
  test eax, eax    <==(5)如果eax=0就跳到44c2dd。让我们看416920
:0044C28F 744C
  je 0044C2DD
----------------------------------------------------------------------------------------------
* Referenced by a CALL at Addresses:
|:0044A5DF  , :0044A80D  , :0044C288  , :0044FC5C  , :0044FE4E
|
:00416920 8B815C020000            mov eax, dword
ptr [ecx+0000025C]  <==(7)[ecx+025C]=[4da334]，
          下bpm 4da334,再看看，
          会有更简单的改法，
          只需改2个字节。
:00416926 C3
  ret              <==(6)有没有搞错，这个call如此简单，倒有5次
              引用，就是它，懒的再看了。
              把8b815c020000改为b80100000090
              广告条没有了。收工。
```

2000.6.15
