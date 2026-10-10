

# Label matrix 5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum453.htm`](https://www.pediy.com/kssd/pediy03/forum453.htm)

http://www.strandware.com/download/lmw50.exe   
发装时可用以下形式的 Activation Code 激活 continue install 按钮   
"C2G2G-2T464-A3623-30819-OHJ62"仅能激活按钮,不能继续安装!   
  
希望高手破完能把过程写出来!以便菜鸟受益!呵呵.... 

  

這個軟件我有 4.0 的正版,今天看到這個貼子,于是把他搞下來想升級,誰知道裝都不讓裝,Kao!   
于是架起 TRW2000,准備跟蹤:   
  
先照 BF 大俠的吩咐,把 Frg3117x.dll 換個名字,結果出來熟悉的畫面:"程序運行在 Demo 方式"!4.0 時就這樣,有狗就是正式版,無狗就是"Demo 方式"!現在就正式開始跟蹤吧,Lets go for fun!   
  
1.用狗的,當然是 bpx DeviceIOControl 了,斷了無數次,每次用 pmodule 返回時都回到 spool32 中,不行,得想第二個辦法!   
2.要在程序開始斷下來,又不愿意用 sendmessage 來攔,良久,實在是想不到什么好的辦法,最后只有用 DeviceIOControl,一斷下來先回到 Spool32 之后就狂按 F10(我是按信 F10 不松,也不知道有多少下,不到一分鐘),突然程序運行了!結束程序,回到 TRW 中!在那個 Call 上按 F9.第一步收工!   
3.再運行程序,在斷點處按 F8 進去(此時,已是 lmw32.exe 的領空),又狂按 F10.........最終你會來到 lmw50v.dll 的領空,成功一半了,呵呵...   
4.這時一定要注意 DS:02124360 的值,開始的時候他是 1.如果他變成 5,嘿嘿.....你就 4 了 4 了的!   
5.知道他不能為 5,那么就 looklook 他為什么等于 5 了,中間跟蹤過程太多,不一一列出,如果你見到以下代碼,你就成功了!   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:10236E26(C)   
|   
:10236E35 0FBE55E8                movsx edx, byte ptr \[ebp\-18\]   
:10236E39 83FA31                  cmp edx, 00000031   
:10236E3C 7553                    jne 10236E91   
:10236E3E E8DCD50100              call 1025441F    //這里出來 EAX 不為 0 完了 !!!   
:10236E43 85C0                    test eax, eax    //如果是 2,肯定你的 Frg3117x.dll 沒有刪!!!也麻煩!   
:10236E45 741B                    je 10236E62        
:10236E47 8B8D5CFDFFFF            mov ecx, dword ptr \[ebp+FFFFFD5C\]   
:10236E4D E8DEF5E1FF              call 10056430   
:10236E52 0C04                    or al, 04   
:10236E54 50                      push eax   
:10236E55 8B8D5CFDFFFF            mov ecx, dword ptr \[ebp+FFFFFD5C\]   
:10236E5B E8F08DE6FF              call 1009FC50   
:10236E60 EB2D                    jmp 10236E8F   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:10236E45(C)   
|   
:10236E62 8B8D5CFDFFFF            mov ecx, dword ptr \[ebp+FFFFFD5C\]   
:10236E68 E8C3F5E1FF              call 10056430    //剛才跳完,來到這里!這個 Call 很簡單,只是把 DS:02124360 中的值給 EAx 見下面的代碼!   
:10236E6D 83E040                  and eax, 00000040 //and 完之后 EAX 為 0,你又完了!   
:10236E70 85C0                    test eax, eax   
:10236E72 751B                    jne 10236E8F      //我跳.........   
:10236E74 8B8D5CFDFFFF            mov ecx, dword ptr \[ebp+FFFFFD5C\]   
:10236E7A E8B1F5E1FF              call 10056430   
:10236E7F 0C04                    or al, 04   
:10236E81 0C40                    or al, 40   
:10236E83 50                      push eax   
:10236E84 8B8D5CFDFFFF            mov ecx, dword ptr \[ebp+FFFFFD5C\]   
:10236E8A E8C18DE6FF              call 1009FC50   
  
call 10056430   
  
:10056430 55                      push ebp   
:10056431 8BEC                    mov ebp, esp   
:10056433 51                      push ecx   
:10056434 894DFC                  mov dword ptr \[ebp\-04\], ecx   
:10056437 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:1005643A 8B8020010000            mov eax, dword ptr \[eax+00000120\]   
:10056440 8BE5                    mov esp, ebp   
:10056442 5D                      pop ebp   
:10056443 C3                      ret   
  
  
經過以上兩跳之后,進入程序,所以的"Demo"字樣已完全不見了,呵呵.....當然,時間,功能什么的就....   
  
6.其中 Call 1025441F 相當之重要,如 ds:02124360 中的值就是在這里面給的,不同的值會有和種不同的結果...改的好的話,Frg3117x.dll 都不用刪!   
7.分析到這里,怎么改各位自已去試了!你可以直接下斷點來到以上的 call 處!   
8.收工!   
  
總結:1.開始進入斷點的方法很愚蠢,希望高手指點一下?   
    2.多謝 BF 大俠發現 Frg3117x.dll,把它刪掉之后我才有信心 Crack 的!BF 大俠能不能賜教你是怎么找到這個秘密的???多謝先!   
    3.這個東東好象不需要狗!? 反正我的 4.0 是要狗的!可能是 5.0 改了吧!   
    4.多謝這位發信息的朋友,我還不知道他升級到了 5.0 呢!呵呵...比 4.0 好用!(廢話)   
號外:發現一個 Hex Workshop 的問題,我用它改 Lmw50v.dll 完成之后,存盤,一切都正常!但是運行時完全反應,,,再跟蹤一看,根本沒改到!!!最后只有用 Save as 用另外的文件名存完后,再換回文件名的方法!問題可能是 windows 沒有釋放這個束 DLL,但 Hex 也應該告訴我不能存啊!   
  
  
iceship   
2000-12-14  4dd466d0e1e0cc123afbe504da2462ca