
```assembly
100732E8 C1 EA 03    shr       edx,3   
100732EB 74 43        je        10073330   
100732ED 89 07        mov       dword ptr [edi],eax   
100732EF 89 47 04    mov       dword ptr [edi+4],eax   
100732F2 4A           dec       edx   
100732F3 74 30        je        10073325   
100732F5 89 47 08    mov       dword ptr [edi+8],eax   
100732F8 89 47 0C    mov       dword ptr [edi+0Ch],eax   
100732FB 4A           dec       edx   
100732FC 74 27        je        10073325   
100732FE 89 47 10    mov       dword ptr [edi+10h],eax   
10073301 89 47 14    mov       dword ptr [edi+14h],eax   
10073304 4A           dec       edx   
10073305 74 1E        je        10073325   
10073307 89 47 18    mov       dword ptr [edi+18h],eax   
1007330A 89 47 1C    mov       dword ptr [edi+1Ch],eax   
1007330D 4A           dec       edx   
1007330E 74 15        je        10073325   
10073310 89 47 20    mov       dword ptr [edi+20h],eax   
10073313 89 47 24    mov       dword ptr [edi+24h],eax   
10073316 4A           dec       edx   
10073317 74 0C        je        10073325   
10073319 89 47 28    mov       dword ptr [edi+28h],eax   
1007331C 89 47 2C    mov       dword ptr [edi+2Ch],eax   
1007331F 8D 7F 30    lea       edi,[edi+30h]   
10073322 4A           dec       edx   
10073323 75 C8        jne       100732ED   
10073325 89 01        mov       dword ptr [ecx],eax   
10073327 89 41 FC    mov       dword ptr [ecx-4],eax   
1007332A 8B 45 08    mov       eax,dword ptr [ebp+8]   
1007332D 5F           pop       edi   
1007332E 5D           pop       ebp   
1007332F C3          ret   
```

```c
}   
*/   
//--------------------------------------------------------------------------   

//   
//       Initialize.   
//   
//       IDA will call this function only once.   
//       If this function returns PLGUIN_SKIP, IDA will never load it again.   
//       If this function returns PLUGIN_OK, IDA will unload the plugin but   
//       remember that the plugin agreed to work with the database.   
//       The plugin will be loaded again if the user invokes it by   
//       pressing the hotkey or selecting it from the menu.   
//       After the second load the plugin will stay on memory.   
//       If this function returns PLUGIN_KEEP, IDA will keep the plugin   
//       in the memory. In this case the initialization function can hook   
//       into the processor module and user interface notification points.   
//       See the hook_to_notification_point() function.   
//   
//-----------------------   
// Do our job here   
//-----------------------   
// length   
#define Sig1      0x4b6a   
#define ChgSig1      0x4c6a   

// Ugly thin border   
#define Sig2      0xc468   

// call instruction byte   
#define Call      0xe8   

// Ugly solid border   
#define Sig3      0xdb68   

// 选择你喜欢的边界线，反注释   
//-------------------------------------   
//#define ThinBorder 0x2d2d968 // ==   
//#define SolidBorder 0x3d3d68 // --   
//-------------------------------------   
// 以下适用于简体中文 Windows!!!   
//-------------------------------------   
//#define ThinBorder 0xa5a968  //━   
//#define SolidBorder 0xfea168 //〓   
//------------------------------------   
#define ThinBorder  0xeea168 //☆   
#define SolidBorder 0xefa168 //★   
//------------------------------------   
//#define ThinBorder 0xaaa168  //—   
//#define SolidBorder 0xf9a168 //※   
//------------------------------------   
// #define ThinBorder 0xf0a168  //○   
// #define SolidBorder 0xf1a168 //●   
//------------------------------------   
// #define ThinBorder 0xf3a168  //◇   
// #define SolidBorder 0xf4a168 //◆   
//------------------------------------   
// #define ThinBorder 0xf5a168  //□   
// #define SolidBorder 0xf6a168 //■   
//------------------------------------   
// #define ThinBorder 0xf7a168  //△   
// #define SolidBorder 0xf8a168 //▲   
//------------------------------------   

typedef bool (*BorderFunc)(void);   

static bool PatchBorderFuncs(void)   
{   
    BorderFunc func;   
    unsigned char* ptr, *ptr2;   
    int i, j;   
    // 获得要 patch 函数的 thunk 地址   
    func = MakeBorder;   
    // 这才是函数在 ida.wll 中的真实地址   
    ptr = (unsigned char**)(char*)func + 2;   
    // 最多搜索 80 个字符   
    for (i = 0; i < 80; i++) {   
        // 找到要 patch 的指令   
        // patch 'push 4bh'   
        if (*(unsigned short*)ptr == Sig1) {    // search for 'push 0c4h'   
            *(unsigned short*)ptr = ChgSig1;   
            ptr += sizeof(short);   
        }   
        // patch 'push c4'   
        if (*(unsigned int*)ptr == Sig2) {   
            *(unsigned int*)ptr = ThinBorder;       // change to 'push 02dh   
            ptr += sizeof(int);   
            // 找到'call sub_100732C4'指令   
            for (ptr2 = ptr; *ptr2 != Call; ptr2++);   
            // 模拟'call'的过程跟踪到 sub_100732C4 地址   
            ptr2 += 5 + *(int*)(ptr2 + 1);   
            // 找到 mov al,byte ptr [ebp+0Ch]   
            for (j = 0; j < 30; j++) {   
                if ((*(unsigned int*)ptr2 & 0xffffff) == 0x0c458a) {   
                    // patch 成 mov eax dword ptr [ebp+0ch]   
                    *ptr2 = 0x8b;   
                    ptr2 += 3;   
                }   
                // 找到 mov ah, al   
                if (*(unsigned short*)ptr2 == 0xe08a) {   
                    // patch 成 nop   
                    *(unsigned short*)ptr2 = 0x9090;   
                    break;   
                }   
                ptr2++;   
            }   
            // patch mov byte ptr [ebp-19h],0 to  mov byte ptr [ebp-18h],0   
            while (*(unsigned int*)ptr != 0xe745c6) ptr++;   
            *(unsigned int*)ptr = 0xe845c6;   
            break;   
        }   
        ptr++;   
    }   
    if (i == 80) return false;   
    func = MakeSolidBorder;   
    ptr = (unsigned char**)(char*)func + 2;   
    for (i = 0; i < 80; i++) {   
        // patch push 4b   
        if (*(unsigned short*)ptr == Sig1) {    // search for 'push 0c4h'   
            *(unsigned short*)ptr = ChgSig1;   
            ptr += sizeof(short);   
        }   
        // patch push db   
        if (*(unsigned int*)ptr == Sig3) {   
            *(unsigned int*)ptr = SolidBorder;       // change to SolidBorder   
            ptr += sizeof(int);   
        }   
        if ((*(unsigned int*)ptr & 0xffffff) == 0xab7d8d) {   
            *(ptr+2) = 0xac;   
            ptr += 3;   
        }   
        if (*(unsigned int*)ptr == 0xdbc045c6) {   
            *(unsigned int*)ptr = 0x20c145c6;   
            ptr += sizeof(int);   
            // patch mov byte ptr [ebp-19h],0 to  mov byte ptr [ebp-18h],0   
            while (*(unsigned int*)ptr != 0xe745c6) ptr++;   
            *(unsigned int*)ptr = 0xe845c6;   
            return true;   
        }   
        ptr++;   
    }   
    return false;   
}   

int init(void)   
{   
    if (PatchBorderFuncs())   
        msg("Border line plugin loaded successfully!");   
    return PLUGIN_SKIP; // never load again   
}   
//--------------------------------------------------------------------------   

//   
//       The plugin method   
//   
//       This is the main function of plugin.   
//   
//       It will be called when the user selects the plugin.   
//   
//               arg - the input argument, it can be specified in   
//                     plugins.cfg file. The default is zero.   
//   
```
