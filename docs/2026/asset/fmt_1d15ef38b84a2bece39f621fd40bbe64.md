
### `a. showtext_screen`
```
`004453E9`  | `50`                   | push eax                                | [[`5D1A58`]] current text addr in mjo buffer
`004453EA`  | `53`                   | push ebx                                |
`004453EB`  | `8D85` `FCFBFFFF`      | lea eax,dword ptr ss:[ebp-404]          |
`004453F1`  | `50`                   | push eax                                |
`004453F2`  | `FF35` `E4CC5200`      | push dword ptr ds:[`52CCE4`]              | `0052CCE4`:`"煨R"`
`004453F8`  | `68` `90065300`        | push polaris_chs.`530690`                 |
`004453FD`  | `E8` `1ED4FFFF`        | call <polaris_chs.sub_442820>           | showtext_screen
```

### `b. move to next text`
```
`0043A750`  | `8B15` `581A5D00`      | mov edx,dword ptr ds:[`5D1A58`]      | edx:&`"1"`
`0043A756`  | `8B0A`                 | mov ecx,dword ptr ds:[edx]         | [edx]:`"1"`
`0043A758`  | `0FBF01`               | movsx eax,word ptr ds:[ecx]        | get_text_len
`0043A75B`  | `83C1` `02`            | add ecx,2                          | move to text
`0043A75E`  | `890A`                 | mov dword ptr ds:[edx],ecx         | [edx]:`"1"`
`0043A760`  | `C3`                   | ret                                |
```

### `c. preshow_text`
```
`00445140`  | `55`               | push ebp                           |
`00445141`  | `8BEC`             | mov ebp,esp                        |
`00445143`  | `81EC` `180C0000`   | sub esp,C18                        |
`00445149`  | `A1` `10A04C00`    | mov eax,dword ptr ds:[`4CA010`]      |
`0044514E`  | `33C5`             | xor eax,ebp                        |
`00445150`  | `8945` `FC`        | mov dword ptr ss:[ebp-4],eax       |
`00445153`  | `8B0D` `E4CC5200`  | mov ecx,dword ptr ds:[`52CCE4`]      | [`52cce4`] text
`00445159`  | `85C9`             | test ecx,ecx                       |
`0044515B`  | `74` `19`           | je polaris_chs.`445176`             |
`0044515D`  | `8039` `00`         | cmp byte ptr ds:[ecx],0            |
`00445160`  | `75` `24`           | jne polaris_chs.`445186`            |
`00445162`  | `C781` `00040000` `00`| mov dword ptr ds:[ecx+400],0       |
`0044516C`  | `C705` `E4CC5200` `00`| mov dword ptr ds:[`52CCE4`],0       | `0052CCE4`:`"杼R"`
`00445176`  | `33C0`             | xor eax,eax                        |
`00445178`  | `8B4D` `FC`         | mov ecx,dword ptr ss:[ebp-4]       |
`0044517B`  | `33CD`             | xor ecx,ebp                        |
`0044517D`  | `E8` `B5520400`     | call <polaris_chs.sub_48A437>      |
`00445182`  | `8BE5`             | mov esp,ebp                        |
`00445184`  | `5D`               | pop ebp                            |
`00445185`  | `C3`               | ret                                |
`00445186`  | `8B15` `581A5D00`   | mov edx,dword ptr ds:[`5D1A58`]      | [[`5D1A58`]] current text addr in mjo buffer
`0044518C`  | `A1` `94CB4D00`     | mov eax,dword ptr ds:[`4DCB94`]      |
`00445191`  | `53`               | push ebx                           |
`00445192`  | `33DB`             | xor ebx,ebx                        |
`00445194`  | `3B02`             | cmp eax,dword ptr ds:[edx]         |
`00445196`  | `74` `1D`           | je polaris_chs.`4451B5`             |
`00445198`  | `53`               | push ebx                           | extra always 0 ?
`00445199`  | `51`               | push ecx                           | buf
`0044519A`  | `E8` `C1D5FFFF`     | call <polaris_chs.sub_442760>      | show_text
```

### `d. memove, copy string to showbuf`
```
`00445BA0`  | `C780` `E8D05200` `01000000`        | mov dword ptr ds:[eax+52D0E8],1    |
`00445BAA`  | `8D80` `E8CC5200`                  | lea eax,dword ptr ds:[eax+52CCE8]  | eax:L`"簀簀簀簀簀簀簀簀簀"`
`00445BB0`  | `8B35` `581A5D00`                  | mov esi,dword ptr ds:[`5D1A58`]      |
`5D1A58`, mjo decrypt text(some of)
`00445BB6`  | `53`                            | push ebx                           | size
`00445BB7`  | `A3` `E4CC5200`                   | mov dword ptr ds:[`52CCE4`],eax      | write `52cce4`
`00445BBC`  | `FF36`                          | push dword ptr ds:[esi]            | src: [esi] mjo decrypt text
`00445BBE`  | `50`                            | push eax                           | dst: `52cce4`, show test
`00445BBF`  | `E8` `BC4E0400`                   | call <polaris_chs.sub_48AA80>      | memmove
`00445BC4`  | `83C4` `0C`                       | add esp,C                          |
`00445BC7`  | `011E`                          | add dword ptr ds:[esi],ebx         |
`00445BC9`  | `5E`                            | pop esi                            |
`00445BCA`  | `5B`                            | pop ebx                            |
`00445BCB`  | `C3`                            | ret                                |
```

根据测试 `a. showtext_screen` 这个位置最适合作为动态 hook 替换文本的位置，显示内容最接近实际显示的，而且修改后的字符串也会被记录到游戏 backlog 里，可供回看文本。其他的 hook 点有些会调用多次、有些文本不全、有些会显示过多字符（如人名，这些会作为语音标识，但不会显示在对话中）。对于显示函数的 hook，总结如下：

|  |  |
| --- | --- |
| 1  2 | [`52CCE4`] current text，貌似没有字节数量什么的，直接替换即可 / / `at` `00442820` [[`5D1A58`]] current text addr in mjo buffer / / but [[`5D1A58`]] pointer at str end with byte `"42 08"` |

写个脚本 hook 验证一下：

```javascript
function hook_showtext()
/*
for investigating the text structure(offset and content)
and substitude text
*/
{
  Interceptor.attach(ptr(g_base + 0x42820), {
    onEnter: function(args)
    {
      var mjo_struct = ptr(g_base + 0XDC350).readPointer();
      var mjo_name = mjo_struct.readAnsiString();
      var mjo_addr_base = mjo_struct.add(0x29 * 4).readPointer();
      var mjo_addr_cur = ptr(g_base + 0x5D1A58 - 0x400000).readPointer().readPointer();

      /* because point at "42 08", go to the start of str buf addr */
      while(mjo_addr_cur.readU8()!=0) mjo_addr_cur = mjo_addr_cur.sub(1);
      mjo_addr_cur = mjo_addr_cur.sub(1)
      while(mjo_addr_cur.readU8()!=0) mjo_addr_cur = mjo_addr_cur.sub(1);
      mjo_addr_cur = mjo_addr_cur.add(1);

      var text_addr = ptr(g_base + 0x52CCE4 - 0x400000).readPointer();
      /* you can replace your own text here */
      var text = text_addr.readAnsiString();
      /* text_addr.writeAnsiString("+0x" + (mjo_addr_cur - mjo_addr_base).toString(16)); */
      console.log(mjo_name, mjo_addr_base, "+0x" + (mjo_addr_cur - mjo_addr_base).toString(16), text);
    },
  });
}
```

`frida -l winterpolaris_hook.js -f Polaris_chs.exe >1.txt` 将脚本注入游戏，由于控制台无法显示 sjis 字符，因此将输出内容重定向到文件中，然后用 sjis 编码查看。得到的内容如下：

![winterpolaris_frida_showtext](https://bbs.kanxue.com/upload/attach/202107/617776_SCSKJ5MWQ4CKJEM.jpg)

|  |  |
| --- | --- |
