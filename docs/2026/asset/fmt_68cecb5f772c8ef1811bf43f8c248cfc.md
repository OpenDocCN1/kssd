
# [原创]水貼：通用的自建調試系統模型

網上很多代碼，自建調試系統，都是基於`WRK`修改的方法，隨便打開一份帶有自建調試系統的源碼，都能看到`M$`程序豬的代碼，所以，這就是為什麼大部分寫驅動的人都把調試系統固定在`XP`上（因為`win7`以上沒有`WRK`作為參考）

觀察一下網上的代碼，我們大可得到一個規律：

凡是有對於`DBGPort`判斷的地方，都把自己從其他地方得到的`DEBUG_OBJECT`替換掉，大家可以去對比他們的代碼。

那麼我們縱觀全系統，對於`DBGPort`清零，我們可以使用`HOOK`模型。這種`HOOK`模式是我第一次在`AGP`高級班的`7.1.1`上第一次出現。大家可以去網上找，一堆盜版。那麼我們怎麼樣的思路來兼容`WIN7`呢？我們查看`IDA`，隨便找一個訪問到`DBGPort`的代碼：

```
_DbgkMapViewOfSection@20 proc near      ; CODE XREF: NtMapViewOfSection(x,x,x,x,x,x,x,x,x,x)+223p

PAGE:0065733E

PAGE:0065733E var_C0          = dword ptr -0C0h
PAGE:0065733E var_BC          = dword ptr -0BCh
PAGE:0065733E var_A8          = dword ptr -0A8h
PAGE:0065733E var_A0          = dword ptr -0A0h
PAGE:0065733E var_9C          = dword ptr -9Ch
PAGE:0065733E var_98          = dword ptr -98h
PAGE:0065733E var_94          = dword ptr -94h
PAGE:0065733E var_90          = dword ptr -90h
PAGE:0065733E ms_exc          = CPPEH_RECORD ptr -18h
PAGE:0065733E arg_0           = dword ptr  8
PAGE:0065733E

PAGE:0065733E                 push    0B0h
PAGE:00657343                 push    offset stru_455C98
PAGE:00657348                 call    __SEH_prolog4
PAGE:0065734D                 mov     edi, edx
PAGE:0065734F                 mov     eax, large fs:124h
PAGE:00657355                 mov     al, [eax+13Ah]
PAGE:0065735B                 xor     ebx, ebx
PAGE:0065735D                 cmp     al, bl
PAGE:0065735F                 jz      loc_657482
PAGE:00657365                 mov     edx, large fs:124h
PAGE:0065736C                 test    byte ptr [edx+280h], 4
PAGE:00657373                 jnz     loc_657482
PAGE:00657379                 cmp     [ecx+0ECh], ebx
```

那麼我們怎麼讓這裡的`DBGPort`是有效的呢？首先，我們通過`Hook`

`NtCreateDebugObject` 得到`OD`創建的`DEBUG_OBJECT`對象，保存到我們驅動的全局變量：

```
PDEBUG_OBJECT g_DebugPort;
g_DebugPort = ReferencedObject;
```

這個時候我們已經擁有了`DBGPort`了，那麼剩下遵守這個規則：

`JNZ`的，就`hook`這個地方，把`0x0EC`設置為`g_DebugPort`讓判斷通過

需要`DBGPort`的地方，就把`g_DebugPort`給它。

給出一個典型的處理例子：

```
__declspec(naked) VOID NewDbgkMapViewOfSection()
{
        __asm
        {
                        //如果不是游戏进程，则直接返回
                        cmp ecx,eprocess_gameprocess
                        jnz _ret
                        //如果游戏进程为0，也直接返回
                        cmp eprocess_gameprocess,0
                        jz _ret
                        //debugport为0，也要返回了
                        cmp g_DebugPort,0
                        jz _ret
                        /*
                        8408f379 3999ec000000    cmp     dword ptr [ecx+0ECh],ebx
                        8408f37f 0f84fd000000    je      nt!DbgkMapViewOfSection+0x144 (8408f482)
                        */
                        push eax
                        mov eax,g_DebugPort
                        mov dword ptr[ecx+0ECh],eax //修改，讓ec偏移有值繞過判斷
                        pop eax
_ret:
                        jmp [DbgkMapViewOfSectionHookZone]
        }
}
```

最後我想對`QQ`群里的這位同學說：

自己只有一個蛋蛋，見到有兩個蛋蛋的人，自然會很驚呆。

![](https://bbs.kanxue.com/upload/attach/201406/42352_zt4mle2bupev5mi.png)

![](https://bbs.kanxue.com/upload/attach/201406/42352_u8m7wir11pbec8h.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [AGP_TP_WIN7.png](javascript:void(0);)

  （353.99kb，282次下载）

- [QQ Photo20140612230440.jpg](javascript:void(0);)

  （67.15kb，298次下载）
