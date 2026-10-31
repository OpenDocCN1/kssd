
# [原创]昨天写的一个`GetProcAddress`

昨天心血来潮就自己写了个，并复习下 PE 哈哈……

## 写作原因

因为要在一个文件中读入一行函数名然后将其找到（我的一个 DLL 库），所以刚开始我用的是 MS 自己的 `GetProcAddress`，发现怎样都成功不了。实在找不到原因，所以就自己写了个并温习了下 PE。（感谢看雪上的 PE 方面的文章！）

刚开始写出来后发现还是成功不了。经过跟踪终于发现原因，并且这个也是 `GetProcAddress` 不可以成功的原因。就是，在一个文件中读入一行字符后在结尾处是 `"\x0A\x00"`，本来应该是没有 `\x0a` 的。比如文件中一行 `buguty` 被读入后就是 `"buguty''\x0a'\x0'"`，本来没有那个换行符的。所以我在第一次计算 `hash` 的时候加了个 `cmp al,0xa`。

后来我改进了下，就是在之前将我的所有读入的字符串（读入的字符串还有参数等信息）进行了一下转换，去掉结尾的 `\x0a`：

```
_asm
{
        push        esi
        mov         esi, pBuff
FindA:
        lodsb
        cmp         al, 0x0a
        jnz         FindA
        xor         al, al
        mov         byte ptr [esi-1], al
        pop         esi
}
```

## 改进后的代码

```
pExploit GetProcAddr(char* pFuncName, HMODULE hDll)
{
        unsigned long hash;
        pExploit        RetVanlue;
        _asm
        {
                mov                 esi, pFuncName
                call                GetHash
                mov                 hash, eax
                ;找到数据目录表        ebx,edi
                mov                 ebx, hDll                                                                ;base to eax
                mov                 edi, [ebx+0x3c]
                mov                 edi, [edi+ebx+0x78]
                add                 edi, ebx                                                                        ;edi==Addr of IMAGE_EXPORT_DIRECTORY
                mov                 edx, [edi+0x20]
                mov                 esi, dword ptr [edx+ebx]
                add                 esi, ebx                                                                        ;esi-->names
                xor                 edx, edx                                                                        ;counter...
                dec                 edx
                mov                 ecx, [edi+0x18]                                                        ;Number of Names of Funcs
                push                ebx
GetByNameLoop:
                dec                 ecx
                inc                 edx
                call                GetHash
                mov                 eax, hash
                cmp                 eax, ebx
                jz                  HashFind
                cmp                 ecx, 0
                jnz                 GetByNameLoop
                jmp                 UnFindAndEnd
HashFind:
                mov                 eax, [edi+0x24]                                                        ;Get AddressOfNameOrdinals's Address
                pop                 ebx
                add                 eax, ebx
                movzx               ax, word ptr [eax+edx*2]
                mov                 edx, [edi+0x1c]
                add                 edx, ebx
                and                 eax, 0x0ffff
                mov                 eax, [edx+eax*4]
                add                 eax, ebx
                jmp                 FindAndEnd
/* GetHashFunc */
GetHash:
                xor                 ebx, ebx
                xor                 eax, eax
HashLoop:
                lodsb
                cmp                 ah, al
                jz                  GetHashRet
                ror                 ebx, 7
                add                 ebx, eax
                jmp                 HashLoop
GetHashRet:
                mov                 eax, ebx
                ret
/* GetHashFuncEnd */
UnFindAndEnd:
                pop                 esi
                xor                 eax, eax
FindAndEnd:
                mov                 RetVanlue, eax
        }
        return RetVanlue;
}
```

这个被加精，出乎意料哈……多谢老大！以后我会更努力的！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
