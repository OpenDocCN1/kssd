
```asm
.text:0041CCD8
        push    offset unk_46AE6C ; char *
.text:0041CCDD                call
  _strncat
.text:0041CCE2
  mov    edi, offset asc_45D620 ; "$"
.text:0041CCE7
              or      ecx, 0FFFFFFFFh
.text:0041CCEA                xor
  eax, eax
.text:0041CCEC
  repne scasb
.text:0041CCEE
    not    ecx
.text:0041CCF0
        sub    edi, ecx
.text:0041CCF2
              mov    esi, edi
.text:0041CCF4                mov
  edx, ecx
.text:0041CCF6
  mov    edi, offset unk_46AE6C
.text:0041CCFB
            or      ecx, 0FFFFFFFFh
.text:0041CCFE                repne
scasb
.text:0041CD00
mov    ecx, edx
.text:0041CD02
      dec    edi
.text:0041CD03
          shr    ecx, 2
.text:0041CD06
              repe movsd
.text:0041CD08
              mov    ecx, edx
.text:0041CD0A                lea
  eax, [esp+0E4h+var_58]
.text:0041CD11
      and    ecx, 3
.text:0041CD14
            push    eax
.text:0041CD15
              repe movsb
.text:0041CD17
              lea    ecx, [esp+0E8h+var_C8]
.text:0041CD1B                push
  ecx
.text:0041CD1C
  call    sub_41D0A0      ;MD5变换，设结果为Result2[16]
.text:0041CD21                add
  esp, 14h

算法第二步完成

/////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////

算法第三步：

.text:0041CD24
  xor    esi, esi        ; 计数器清零
.text:0041CD26
.text:0041CD26 loc_41CD26:
              ; CODE XREF: sub_41CAD0+328j
.text:0041CD26                lea
  edx, [esp+0D8h+var_B0]
.text:0041CD2A
      push    edx
.text:0041CD2B
          call    sub_41CF80
;MD5初始化
.text:0041CD30
mov    ebx, esi
.text:0041CD32
      add    esp, 4
.text:0041CD35
            and    ebx, 1
.text:0041CD38
              jz      short
loc_41CD4F
.text:0041CD3A
  mov    edi, ebp
.text:0041CD3C
        or      ecx, 0FFFFFFFFh
.text:0041CD3F
              xor    eax, eax
.text:0041CD41                repne scasb
.text:0041CD43                not
  ecx
.text:0041CD45
  dec    ecx
.text:0041CD46
      lea    eax, [esp+0D8h+var_B0]
.text:0041CD4A
              push    ecx
.text:0041CD4B
              push    ebp
.text:0041CD4C
              push    eax
.text:0041CD4D
              jmp    short loc_41CD5B
; Update(Result2,16)
.text:0041CD4F ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0041CD4F
.text:0041CD4F loc_41CD4F:
                    ; CODE XREF:
sub_41CAD0+268j
.text:0041CD4F
    lea    ecx, [esp+0D8h+var_C8]
.text:0041CD53
              push    10h
.text:0041CD55
              lea    edx, [esp+0DCh+var_B0]
.text:0041CD59                push
  ecx
.text:0041CD5A
  push    edx
.text:0041CD5B
.text:0041CD5B loc_41CD5B:
  ; CODE XREF: sub_41CAD0+27Dj
.text:0041CD5B
        call    sub_41CFB0      ;
Update(Result2,16)
.text:0041CD60
    mov    eax, esi
.text:0041CD62
          mov    ecx, 3
.text:0041CD67
              cdq
.text:0041CD68
              idiv    ecx
.text:0041CD6A
              add    esp, 0Ch
.text:0041CD6D                test
  edx, edx
.text:0041CD6F
  jz      short loc_41CD89 ; 余数为零则跳
.text:0041CD71
              mov    edx, [esp+0D8h+var_B4]
.text:0041CD75                mov
  eax, dword_46AEE8
.text:0041CD7A
      push    edx
.text:0041CD7B
          lea    ecx, [esp+0DCh+var_B0]
.text:0041CD7F                push
  eax
.text:0041CD80
  push    ecx
.text:0041CD81
      call    sub_41CFB0      ; Update(
用户名(前8字节，如果用户名长度小于8字节，则用整个用户名),ULONG nInputLen )
.text:0041CD86
            add    esp, 0Ch
.text:0041CD89
.text:0041CD89 loc_41CD89:
              ; CODE XREF: sub_41CAD0+29Fj
.text:0041CD89                mov
  eax, esi
.text:0041CD8B
  mov    ecx, 7
.text:0041CD90
        cdq
.text:0041CD91
        idiv    ecx
.text:0041CD93
            test    edx, edx
.text:0041CD95
              jz      short
loc_41CDB2 ; 余数为零则跳
.text:0041CD97
    mov    edi, ebp
.text:0041CD99
          or      ecx, 0FFFFFFFFh
.text:0041CD9C                xor
  eax, eax
.text:0041CD9E
  lea    edx, [esp+0D8h+var_B0]
.text:0041CDA2
            repne scasb
.text:0041CDA4
              not    ecx
.text:0041CDA6
              dec    ecx
.text:0041CDA7
              push    ecx
.text:0041CDA8
              push    ebp
.text:0041CDA9
              push    edx
.text:0041CDAA
              call    sub_41CFB0
;Update(用户名，用户名长度)
.text:0041CDAF
    add    esp, 0Ch
.text:0041CDB2
.text:0041CDB2
loc_41CDB2:
        ; CODE XREF: sub_41CAD0+2C5j
.text:0041CDB2
              test    ebx, ebx
.text:0041CDB4                jz
    short loc_41CDC4
.text:0041CDB6
        lea    eax, [esp+0D8h+var_C8]
.text:0041CDBA
              push    10h
.text:0041CDBC
              lea    ecx, [esp+0DCh+var_B0]
.text:0041CDC0                push
  eax
.text:0041CDC1
  push    ecx
.text:0041CDC2
      jmp    short loc_41CDD7
.text:0041CDC4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0041CDC4
.text:0041CDC4 loc_41CDC4:
                    ; CODE XREF:
sub_41CAD0+2E4j
.text:0041CDC4
    mov    edi, ebp
.text:0041CDC6
          or      ecx, 0FFFFFFFFh
.text:0041CDC9                xor
  eax, eax
.text:0041CDCB
  lea    edx, [esp+0D8h+var_B0]
.text:0041CDCF
            repne scasb
.text:0041CDD1
              not    ecx
.text:0041CDD3
              dec    ecx
.text:0041CDD4
              push    ecx
.text:0041CDD5
              push    ebp
.text:0041CDD6
              push    edx
.text:0041CDD7
.text:0041CDD7 loc_41CDD7:
              ; CODE XREF: sub_41CAD0+2F2j
.text:0041CDD7                call
  sub_41CFB0      ;Update(Result2,16)/Update(用户名，用户长度)
.text:0041CDDC                add
  esp, 0Ch
.text:0041CDDF
  lea    eax, [esp+0D8h+var_B0]
.text:0041CDE3
            lea    ecx, [esp+0D8h+var_C8]
.text:0041CDE7                push
  eax
.text:0041CDE8
  push    ecx
.text:0041CDE9
      call    sub_41D0A0    ;MD5变换，设结果为Result3[16],此结果即是下一次循环中的Result2[16]
.text:0041CDEE                add
  esp, 8
.text:0041CDF1
  inc    esi
.text:0041CDF2
      cmp    esi, 3E8h
.text:0041CDF8
            jl      loc_41CD26

算法第三步完成

///////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////

算法第四步：将第三步结果转换为规则的可见字符串，长度变为22字节

变换表为："./0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

.text:0041CDFE                mov
  edi, offset unk_46AE6C
.text:0041CE03
      or      ecx, 0FFFFFFFFh
.text:0041CE06
              xor    eax, eax
.text:0041CE08                xor
  edx, edx
.text:0041CE0A
  mov    dh, byte ptr [esp+0D8h+var_C8]
.text:0041CE0E
              push    4
.text:0041CE10
              repne scasb
.text:0041CE12
              mov    eax, [esp+0DCh+var_C4+2]
.text:0041CE16                and
  eax, 0FFh
.text:0041CE1B
    or      edx, eax
.text:0041CE1D
            mov    eax, [esp+0DCh+var_BC]
.text:0041CE21                not
  ecx
.text:0041CE23
  shl    edx, 8
.text:0041CE26
        and    eax, 0FFh
.text:0041CE2B
              dec    ecx
.text:0041CE2C
              or      edx, eax
.text:0041CE2E                add
  ecx, offset unk_46AE6C
.text:0041CE34
      push    edx
.text:0041CE35
          push    ecx
.text:0041CE36
              mov    dword_46AE68,
ecx
.text:0041CE3C
call    sub_41D9B0
.text:0041CE41
      mov    ecx, [esp+0E4h+var_C8+1]
.text:0041CE45
              mov    edx, [esp+0E4h+var_C4+3]
.text:0041CE49                mov
  eax, dword_46AE68
.text:0041CE4E
      and    ecx, 0FFh
.text:0041CE54
            shl    ecx, 8
.text:0041CE57
              and    edx, 0FFh
```
