
SDPI:0047B0E5 `mov byte ptr [eax], 0E9h`

SDPI:0047B0E8 `xor eax, eax`

SDPI:0047B0EA `mov [ecx+4], eax` ; 清除硬件断点

SDPI:0047B0ED `mov [ecx+8], eax`

SDPI:0047B0F0 `mov [ecx+0Ch], eax`

SDPI:0047B0F3 `mov [ecx+10h], eax`

SDPI:0047B0F6

SDPI:0047B0F6 `locret_47B0F6:` ; CODE XREF: SDPI:0047B0CE j

SDPI:0047B0F6 `retn`

SDPI:0047B0F7 ; ----------------------------------------------------------------------------

SDPI:0047B0F7

SDPI:0047B0F7 `pass_47B0F7:` ; CODE XREF: SDPI:0047B0AC j

SDPI:0047B0F7 ; SDPI:0047B0AE j

SDPI:0047B0F7 `pop eax`

SDPI:0047B0F8 `call CallGetTickCount` ; 注意一点的话就会发现和前

SDPI:0047B0F8 ; 一个 INT3 的代码基本上是一样的

SDPI:0047B0F8 ; 作者很喜欢作宏?

SDPI:0047B0F8 ; ----------------------------------------------------------------------------

SDPI:0047B0FD `a@V db 0,10h,'@',0,'綱',1`

SDPI:0047B105 ; ----------------------------------------------------------------------------

SDPI:0047B105

SDPI:0047B105 `loc_47B105:` ; CODE XREF: SDPI:0047B303 p

SDPI:0047B105 `pop ebp`

SDPI:0047B106 `pop eax`

SDPI:0047B107 `jmp near ptr unk_47B308`

SDPI:0047B10C ; ----------------------------------------------------------------------------

SDPI:0047B10C `mov ecx, 0FFFFFF00h`

SDPI:0047B111 `push fs`

SDPI:0047B111 ; ----------------------------------------------------------------------------

SDPI:0047B113 `aTU@I db 't',0Ah`

SDPI:0047B113 `db 'u',8,0,10h,'@',0,'皦?',4`

SDPI:0047B11F ; ----------------------------------------------------------------------------

SDPI:0047B11F `pushfw`

SDPI:0047B121 `push eax`

SDPI:0047B122 `mov eax, ebx`

SDPI:0047B124 `push ebx`

SDPI:0047B125 `mov eax, ecx`

SDPI:0047B127 `push eax`

SDPI:0047B128 `add eax, edx`

SDPI:0047B12A `mov ebx, eax`

SDPI:0047B12C `push ebx`

SDPI:0047B12D `pop eax`

SDPI:0047B12E `push edx`

SDPI:0047B12F `call loc_47B13C`

SDPI:0047B12F ; ----------------------------------------------------------------------------

SDPI:0047B134 `dd 401000h`

SDPI:0047B138 `dd 132BD7B0h`

SDPI:0047B13C ; ----------------------------------------------------------------------------

SDPI:0047B13C

SDPI:0047B13C `loc_47B13C:` ; CODE XREF: SDPI:0047B12F p

SDPI:0047B13C `pop eax`

SDPI:0047B13D `call loc_47B143`

SDPI:0047B142 `nop`

SDPI:0047B143

SDPI:0047B143 `loc_47B143:` ; CODE XREF: SDPI:0047B13D p

SDPI:0047B143 `pop eax`

SDPI:0047B144 `add eax, 11h`

SDPI:0047B149 `push eax`

SDPI:0047B14A `jmp GetTickCount`

SDPI:0047B14A ; ----------------------------------------------------------------------------

SDPI:0047B14F `db 90h` ; ?

SDPI:0047B150 `db 90h` ; ?

SDPI:0047B151 `db 90h` ; ?

SDPI:0047B152 `db 90h` ; ?

SDPI:0047B153 ; ----------------------------------------------------------------------------

SDPI:0047B153 `push eax`

SDPI:0047B154 `mov eax, edx`

SDPI:0047B156 `push eax`

SDPI:0047B157 `call loc_47B15D`

SDPI:0047B15C `nop`

SDPI:0047B15D

SDPI:0047B15D `loc_47B15D:` ; CODE XREF: SDPI:0047B157 p

SDPI:0047B15D `pop edx`

SDPI:0047B15E `add edx, 52h`

SDPI:0047B164 `push edx`

SDPI:0047B165 `add edx, 402165h`

SDPI:0047B16B `push edx`

SDPI:0047B16C `jo short loc_47B1C1`

SDPI:0047B16E `jno short loc_47B1C1`

SDPI:0047B170

SDPI:0047B170 ; ****** S U B R O U T I N E ******

SDPI:0047B170

SDPI:0047B170

SDPI:0047B170 `sub_47B170 proc near` ; CODE XREF: SDPI:0047B1B4 p

SDPI:0047B170 `pop eax`

SDPI:0047B171 `pop ebx`

SDPI:0047B172 `call sub_47B178`

SDPI:0047B177 `nop`

SDPI:0047B177 `sub_47B170 endp`

SDPI:0047B177

SDPI:0047B178

SDPI:0047B178 ; ****** S U B R O U T I N E ******

SDPI:0047B178

SDPI:0047B178

SDPI:0047B178 `sub_47B178 proc near` ; CODE XREF: sub_47B170+2 p

SDPI:0047B178 `pop eax`

SDPI:0047B179 `add eax, 11h`

SDPI:0047B17E `push eax`

SDPI:0047B17F `jmp GetTickCount`

SDPI:0047B17F `sub_47B178 endp`

SDPI:0047B17F

SDPI:0047B17F ; ----------------------------------------------------------------------------

SDPI:0047B184 `db 90h` ; ?

SDPI:0047B185 `db 90h` ; ?

SDPI:0047B186 `db 90h` ; ?

SDPI:0047B187 `db 90h` ; ?

SDPI:0047B188 ; ----------------------------------------------------------------------------

SDPI:0047B188 `pop ebx`

SDPI:0047B189 `add ebx, 1F4h`

SDPI:0047B18F `sub ebx, eax`

SDPI:0047B191 `js short OVER_47B1D5` ; 同前一个是一样的,不能跳

SDPI:0047B193 `call loc_47B199`

SDPI:0047B198 `nop`

SDPI:0047B199

SDPI:0047B199 `loc_47B199:` ; CODE XREF: SDPI:0047B193 p

SDPI:0047B199 `pop ebx`

SDPI:0047B19A `add ebx, 0A5h`

SDPI:0047B1A0 `push ebx`

SDPI:0047B1A1 `call loc_47B1CB`

SDPI:0047B1A1 ; ----------------------------------------------------------------------------

SDPI:0047B1A6 `dd 401000h`

SDPI:0047B1AA `dd 58C88B0h`

SDPI:0047B1AE ; ----------------------------------------------------------------------------

SDPI:0047B1AE `pop eax`

SDPI:0047B1AF `mov edx, eax`

SDPI:0047B1B1 `mov eax, ebx`

SDPI:0047B1B3 `push eax`

SDPI:0047B1B4 `call sub_47B170`

SDPI:0047B1B4 ; ----------------------------------------------------------------------------

SDPI:0047B1B9 `dd 401000h`

SDPI:0047B1BD `dd 1833639h`

SDPI:0047B1C1 ; ----------------------------------------------------------------------------

SDPI:0047B1C1

SDPI:0047B1C1 `loc_47B1C1:` ; CODE XREF: SDPI:0047B16C j

SDPI:0047B1C1 ; SDPI:0047B16E j

SDPI:0047B1C1 `pop eax`

SDPI:0047B1C2 `retn`

SDPI:0047B1C2 ; ----------------------------------------------------------------------------

SDPI:0047B1C3 `JUNK_47B1C3 db 0,10h,'@',0,'>V|',7`
