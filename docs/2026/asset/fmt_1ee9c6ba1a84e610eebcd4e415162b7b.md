
```
00007d5f: (                    ) : mov al, byte ptr ds:0x1f8 ; a0f801
00007d62: (                    ) : call .+0x0008             ; e80800       call PrintErrorMsg
00007d65: (                    ) : mov al, byte ptr ds:0x1fb ; a0fb01
00007d68: (                    ) : call .+0x0002             ; e80200       call PrintErrorMsg
00007d6b: (                    ) : jmp .+0xfffe              ; ebfe         hlt
```

### `PrintErrorMsg`

```
00007d6d: (                    ) : mov ah, 0x01              ; b401
00007d6f: (                    ) : mov si, ax                ; 8bf0
00007d71: (                    ) : lodsb al, byte ptr ds:[si] ; ac
00007d72: (                    ) : cmp al, 0x00              ; 3c00
00007d74: (                    ) : jz .+0x0009               ; 7409
00007d76: (                    ) : mov ah, 0x0e              ; b40e
00007d78: (                    ) : mov bx, 0x0007            ; bb0700
00007d7b: (                    ) : int 0x10                  ; cd10
00007d7d: (                    ) : jmp .+0xfff2              ; ebf2
00007d7f: (                    ) : ret                       ; c3
```

总结DBR，拷贝`Hidden sectors + 1`之后的16个Sector到`7e00`开始扇区中，然后跳转到`0x7e7a`处开始执行。

## `BOOTMGR` 分析

```
00007e7a: (                    ) : movzx eax, word ptr ds:0xb ; 660fb7060b00    Bytes per sector
00007e80: (                    ) : movzx ebx, byte ptr ds:0xd ; 660fb61e0d00    Sectors per cluster
00007e86: (                    ) : mul eax, ebx              ; 66f7e3           计算每个蔟的字节数
00007e89: (                    ) : mov dword ptr ds:0x252, eax ; 66a35202       dword ptr[0x7e52] < eax
00007e8d: (                    ) : mov ecx, dword ptr ds:0x40 ; 668b0e4000      FILE record size indicator
00007e92: (                    ) : cmp cl, 0x00              ; 80f900
00007e95: (                    ) : jnle .+0x000e             ; 0f8f0e00          jnle 00007ea7
00007e99: (                    ) : neg cl                    ; f6d9
00007e9b: (                    ) : mov eax, 0x00000001       ; 66b801000000
00007ea1: (                    ) : shl eax, cl               ; 66d3e0            0x400
00007ea4: (                    ) : jmp .+0x0008              ; eb08              jmp 00007eae
00007ea6: (                    ) : nop                       ; 90
00007ea7: (                    ) : mov eax, dword ptr ds:0x252 ; 66a15202
00007eab: (                    ) : mul eax, ecx              ; 66f7e1             计算File Record Entry的大小
                                                            ;                    (字节为单位)
00007eae: (                    ) : mov dword ptr ds:0x266, eax ; 66a36602
00007eb2: (                    ) : movzx ebx, word ptr ds:0xb ; 660fb71e0b00      Bytes per sector
00007eb8: (                    ) : xor edx, edx              ; 6633d2
00007ebb: (                    ) : div eax, ebx              ; 66f7f3
00007ebe: (                    ) : mov dword ptr ds:0x256, eax ; 66a35602
00007ec2: (                    ) : call .+0x0495             ; e89504             call 0000835a
```

偶现在也就分析到这儿了，以后再续吧

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
