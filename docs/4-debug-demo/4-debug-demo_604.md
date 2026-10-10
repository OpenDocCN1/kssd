

# NetAnts1.2

> 原文：[`www.pediy.com/kssd/pediy02/forum189.htm`](https://www.pediy.com/kssd/pediy02/forum189.htm)

 希望各位大虾能够指点一下小弟！   
网络蚂蚁的广告条是如何实现和如何破解的！  

\*netants1.22   
  
netants.exe   
  
去掉广告   
  
8345F443   
90909090   
  
6a42   
6a00   
.   
.   
push 42  ->nop   
.   
. 

  

 假使你知道 NetAnts1.2 是用 Movewindow 來調用廣告視窗的話,你可以下 bpx movewindow,等攔下後返回,會來到:   
:0041AA4D push EDI   
:0041AA4E CALL 00402452 <--調用函數的 CALL   
:0041AA53 CMP EAX,EBX <--我們在這   
:0041AA55 POP ECX   
:0041AA56 MOV \[00446888\],EAX   
:0041AA5B JZ 0041C8E3   
:0041AA61 PUSH EDI   
若你不知道用 MoveWindow,可下 bpx Createwindowexa 令其返回值等於 0 看看那一個窗口不見了,如此 Try and Error 你會發現上述的 CODE 是在處理廣告視窗   
分析一下 CODE 你會發現程式會將窗口的 ID 放入 00446888 中供程式應用,那我們   
要怎麼改,當然不要讓它產生 ID 和不影響程式的運作,我的改法是:   
:0041AA4d push EDI -->XOR EAX,EAX   
:0041AA4E CALL 00402452 -->NOP 掉   
:0041AA53 CMP EAX,EBX -->NOP 掉   
:0041AA55 POP ECX -->NOP 掉   
:0041AA56 MOV \[00446888\],EAX -->保留   
:0041AA5B JZ 0041C8E3 -->NOP 掉   
這樣改了以後還是有視窗,只是沒有畫面而已.沒關係我們用 W32Dasm 反匯編(在台灣叫反組譯)查找 00446888 亦就是廣告視窗的 Flag,我找到:   
:0041A80E PUSH DWORD PTR \[00446888\]   
:0041A814 CALL EDI <--\[USER32!MoveWindow\]   
:0041A816 ADD DWORD PTR \[EBP\-0C\],00000043   
:0041A81A ADD DWORD PTR \[00444E2C\],00000043   
:0041A821 CMP DWORD PTR \[00446A90\],EBX   
有編程經驗的人大概知道這是在設定窗口的大小,我們用 Symbole Loader 載入直接 G 到此處,結果發現程式在 SetWindowPlacement,那我們應該知道怎麼改了   
就是將 0041A816 和 0041A81A 的 00000043 改為 00000000 改完之後執行程式,清爽多了吧!這是 1.2 版的其他版本應該差不多吧!我儘量寫的讓大家看的懂且我不是什麼高手,水平很低的只是經驗分享而已,談不上教程,所以有表達不善之處,還請見諒!!!  

 由于我现在安装的是 Win2000，SoftICE 不太好使，而且我喜欢 NetVampire，对 NetAnt 不太注意，所以不解 1.22 版了，以前解过一个版本，原理应该差不多，凑合着看吧。   
The MoveWindow function changes the position and dimensions of the specified window. For a top-level window, the position and dimensions are relative to the upper-left corner of the screen. For a child window, they are relative to the upper-left corner of the parent window's client area.   
BOOL MoveWindow(   
  
    HWND hWnd,    // handle of window   
    int X,    // horizontal position   
    int Y,    // vertical position   
    int nWidth,    // width   
    int nHeight,    // height   
    BOOL bRepaint     // repaint flag   
  );   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00417F4C(C)   
|   
:00417F88 891D80BD4300            mov dword ptr \[0043BD80\], ebx   
:00417F8E 891D88BD4300            mov dword ptr \[0043BD88\], ebx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00417F86(U)   
|   
:00417F94 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
改为:   
00417F94 EB1790   
（此处跳到 00417FAD，就是跳过 00417FAB FFD7  call edi 这个 CALL 是调广告窗口的）          
  
\* Possible Reference to String Resource ID=00001: "mailto:support@netants.com?subject=Feedback of "   
                                  |   
:00417F97 6A01                    push 00000001   
:00417F99 2B45F0                  sub eax, dword ptr \[ebp\-10\]   
:00417F9C 6A3F                    push 0000003F   
:00417F9E 50                      push eax   
:00417F9F FF75F4                  push \[ebp\-0C\]   
:00417FA2 FF75F0                  push \[ebp\-10\]   
:00417FA5 FF356CD44300            push dword ptr \[0043D46C\]   
:00417FAB FFD7                    call edi（调用 movewindow 函数）   
:00417FAD 8345F440                add dword ptr \[ebp\-0C\],00000040   
改为:   
00417FAD 8345F400   
(00417FAD 那行加上的 00000040 就是广告条的高度，将其改为加上 0 即可）   
这只是一部分破解，前边还有一处是调用 adver.dll 的，很好改，在 w32dasm 中的 string 找到 adver.dll 将其跳过即可）   
其实和 C-pen 讲的一样，很欢迎台湾的朋友。   4dd466d0e1e0cc123afbe504da2462ca