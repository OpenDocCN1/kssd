
## 汇编代码
```
.text:004300AC
.text:004300AC TO_BIG_ENDIAN:                  ; CODE XREF: sha1_final+29j
.text:004300AC              mov     edx, [_ctx+eax*4+SHA1_CTX.count0]
.text:004300B0              shr     edx, cl             ; ctx->count[eax] >> ((3-(i&3))*8)
.text:004300B2              and     dl, 0FFh        ; (ctx->count[eax] >> ((3-(i&3))*8)) & 0x0ff;
.text:004300B5              mov     ecx, [esi]      ; i
.text:004300B7              mov     byte ptr [esp+ecx+24h+bigcount], dl
.text:004300BB              inc     dword ptr [esi] ; i++
.text:004300BD              cmp     dword ptr [esi], 8 ; i<8
.text:004300C0              jb      short NEXT_ENDIAN ; 处理字节序, 把 ctx 中的 count[0],count\1 数据 bit 数
.text:004300C0                                    ; 转化为 BIG_ENDIAN 存储到 bigcount[8]
.text:004300C0
.text:004300C2              mov     byte ptr [esp+24h+end], 80h
.text:004300C7              mov     al, byte ptr [esp+24h+_trans_copies]
.text:004300CA              mov     ecx, 1          ; len
.text:004300CF              push    eax             ; trans_copies
.text:004300D0              mov     eax, _ctx       ; ctx
.text:004300D2              lea     edx, [esp+28h+end] ; unsigned int
.text:004300D6              call    sha1_update     ; sha1_update 一个 bit 数据 0x80, 这也许是表示数据的结束?
.text:004300D6
.text:004300DB              jmp     short FOR_condition
.text:004300DD ; ---------------------------------------------------------------------------
.text:004300DD FOR:                                  ; CODE XREF: sha1_final+83j
.text:004300DD              mov     byte ptr [esp+14h], 0 ; end
.text:004300E2              mov     dl, [esp]       ; _trans_copies
.text:004300E5              mov     ecx, 1          ; data
.text:004300EA              push    edx             ; trans_copies
.text:004300EB              lea     edx, [esp+18h]  ; end
.text:004300EF              mov     eax, _ctx       ; ctx
.text:004300F1              call    sha1_update
.text:004300F1
.text:004300F6
.text:004300F6 FOR_condition:                       ; CODE XREF: sha1_final+5Bj
.text:004300F6              mov     eax, [_ctx+SHA1_CTX.count0]
.text:004300F9              and     eax, 1F8h       ; 0x1F8 = 504 = 63*8
.text:004300FE              cmp     eax, 1C0h       ; 0x1c0 = 488 = 56*8
.text:00430103              jnz     short FOR       ; 判断剩余的 bit 数是不是等于 488(56*8, 字节数是不是等
.text:00430103                                    ; 于 56), 不等于添加'\0', 直到等于 56, 剩余 8 个字节添加
.text:00430103                                    ; bigcount[8], 凑够 64 字节, 计算.
.text:00430103
.text:00430105              mov     dl, [esp]       ; _trans_copies
.text:00430108              mov     ecx, 8          ; data
.text:0043010D              push    edx             ; trans_copies
.text:0043010E              lea     edx, [esp+10h]  ; bigcount
.text:00430112              mov     eax, _ctx       ; ctx
.text:00430114              call    sha1_update     ; 把 bigcount[8]添加到 ctx->buffer, 计算最后 64 字节,
.text:00430114                                    ; 整个计算过程结束
.text:00430114
.text:00430119              xor     eax, eax
.text:0043011B              mov     [esi], eax      ; i = 0
.text:0043011B
.text:0043011D
.text:0043011D OUT_DIGEST:                          ; CODE XREF: sha1_final+AFj
.text:0043011D              mov     edx, [esi]
.text:0043011F              mov     ecx, [esi]
.text:00430121              mov     eax, [_ctx+ecx*4+SHA1_CTX.state]
.text:00430124              and     eax, 0FFFFFFFFh
.text:00430127              mov     [_digest+edx*4], eax
.text:0043012A              inc     dword ptr [esi] ; i++
.text:0043012C              cmp     dword ptr [esi], 5
.text:0043012F              jb      short OUT_DIGEST ; 输出 sha1 摘要的时候是按 DWORD 输出, LITTLE_ENDIA 字节徐
.text:0043012F
.text:00430131              push    4               ; n
.text:00430133              push    0               ; c
.text:00430135              push    esi
.text:00430136              call    _memset         ; i 清零
.text:00430136
.text:0043013B              add     esp, 0Ch
.text:0043013E              push    4               ; n
.text:00430140               push    0               ; c
.text:00430142               lea     edx, [esp+10h]
.text:00430146               push    edx             ; s
.text:00430147               call    _memset         ; var_1C 清零
.text:00430147
.text:0043014C               add     esp, 0Ch
.text:0043014F               lea     esi, [_ctx+SHA1_CTX.buffer]
.text:00430152               push    40h             ; n
.text:00430154               push    0               ; c
.text:00430156               push    esi             ; s
.text:00430157               call    _memset         ; ctx->buffer 清零
.text:00430157
.text:0043015C               add     esp, 0Ch
.text:0043015F               push    14h             ; n
.text:00430161               push    0               ; c
.text:00430163               push    _ctx            ; s
.text:00430164               call    _memset         ; ctx->state 清零
.text:00430164
.text:00430169               add     esp, 0Ch
.text:0043016C               lea     eax, [_ctx+SHA1_CTX.count0]
.text:0043016F               push    8               ; n
.text:00430171               push    0               ; c
.text:00430173               push    eax             ; s
.text:00430174               call    _memset         ; ctx->count[0], ctx->count[1]清零
.text:00430174
.text:00430179               add     esp, 0Ch
.text:0043017C               push    8               ; n
.text:0043017E               push    0               ; c
.text:00430180               lea     edx, [esp+20h+s]
.text:00430184               push    edx             ; s
.text:00430185               call    _memset         ; bigcount 清零
.text:00430185
.text:0043018A               add     esp, 0Ch
.text:0043018D               cmp     byte ptr [esp], 0 ; _trans_copies
.text:00430191               jz      short loc_43019E
.text:00430191
.text:00430193               mov     edx, esi        ; data
.text:00430195               mov     cl, 1           ; trans_copies
.text:00430197               mov     eax, _ctx       ; ctx_state
.text:00430199               call    sha1_transform  ;/* ctx 中产生一个坏的 SHA1 摘要信息 */
.text:00430199
.text:0043019E loc_43019E:                           ; CODE XREF: sha1_final+111j
.text:0043019E               add     esp, 18h
.text:004301A1               pop     _digest
.text:004301A2               pop     esi
.text:004301A3               pop     _ctx
.text:004301A4               retn
.text:004301A4 sha1_final      endp ; sp =  0Ch
```

## C 实现
```
void __fastcall
sha1_final(SHA1_CTX *ctx, u_int32_t *digest, u_int8_t trans_copies)
{
        int i;
        int var_1C;                /* ???? 没用到的变量 */
        u_int8_t bigcount[8];
        u_int8_t end;
        for (i = 0; i < 8; i++)        /* 字节序转换为 BIG_ENDIA */
        {
                bigcount[i] = (u_int8_t) ((ctx->count[(i >= 4 ? 0 : 1)] >> ((3 - (i & 3)) * 8)) & 255);
        }
        end = 0x80;
        sha1_update(ctx,  &end, 1, trans_copies);
        for (; (ctx->count[0] & 504) != 448; )
        {
                end = 0;
                sha1_update(ctx, &end, 1, trans_copies);
        }
}
```
