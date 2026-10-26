
Find

`1F02A5E8=1F02A5E8` (ASCII `"Visual Assist X is loaded but dormant. You should uninstall the "`,LF,`"software or purchase a license if your trial has expired."`)

`1ED4EE4B`|.`E8 80111700`   `call`    `1EEBFFD0`     ////`enter`     `33 C0 40 C3`               Repalcer

`1ED4EE50`|.`83C4 04`       `add`     `esp`, 4

`1ED4EE53`|.`85C0`          `test`    `eax`, `eax`

`1ED4EE55`|.`75 2B`         `jnz`     `short` `1ED4EE82`

`1ED4EE57`|.`8B15 000A0F1F` `mov`     `edx`, `dword` `ptr` `[1F0F0A00]`

`1ED4EE5D`|.`50`            `push`    `eax`                              `; /Style`

`1ED4EE5E`|.`68 A44D021F`   `push`    `1F024DA4`                         `; |Title = "Visual Assist X"`

`1ED4EE63`|.`68 E8A5021F`   `push`    `1F02A5E8`                         `; |Text = "Visual Assist X is loaded but dormant. You should uninstall the ",LF,"software or purchase a license if your trial has expired."`

`1ED4EE68`|.`52`            `push`    `edx`                              `; |hOwner => NULL`

`1ED4EE69`|.`FF15 143A021F` `call`    `dword` `ptr` `[<&user32.MessageBoxA>>` `; \MessageBoxA`

like this

`1EEBFFD0`      `33C0`          `xor`     `eax`, `eax`

`1EEBFFD2`      `40`            `inc`     `eax`

`1EEBFFD3`      `C3`            `retn`

`1EEBFFD4`   `?`  `46`            `inc`     `esi`

`1ED4EE93`|.`68 D8A5021F`   `push`    `1F02A5D8`                         `;  ASCII "CLOCKBACK"`

`1ED4EE98`|.`50`            `push`    `eax`

`1ED4EE99`|.`E8 42FAFFFF`   `call`    `1ED4E8E0`

`1ED4EE9E`|.`83C4 08`       `add`     `esp`, 8

`1ED4EEA1`|.`C64424 50 15`  `mov`     `byte` `ptr` `[[esp]+50]`, 15

`1ED4EEA6`|.`8B08`          `mov`     `ecx`, `dword` `ptr` `[[eax]]`

`1ED4EEA8`|.`8B71 F4`       `mov`     `esi`, `dword` `ptr` `[[ecx]-C]`

`1ED4EEAB`|.`885C24 50`     `mov`     `byte` `ptr` `[[esp]+50]`, `bl`

`1ED4EEAF`|.`8D4C24 38`     `lea`     `ecx`, `dword` `ptr` `[[esp]+38]`

`1ED4EEB3`|.`E8 9828FBFF`   `call`    `1ED01750`

`1ED4EEB8`|.`85F6`          `test`    `esi`, `esi`

`1ED4EEBA`|.`74 29`         `je`      `short` `1ED4EEE5`                        //eb

`1ED4EEBC`|.`8B15 000A0F1F` `mov`     `edx`, `dword` `ptr` `[1F0F0A00]`

`1ED4EEC2`|.`52`            `push`    `edx`

`1ED4EEC3`|.`6A 00`         `push`    `0`

`1ED4EEC5`|.`68 10A5021F`   `push`    `1F02A510`                         `;  ASCII "Visual Assist X is unable to start your trial. You may need to reboot your system. If the problem persists, contact us at c7bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6Z5L8$3I4W2N6r3!0E0j5i4c8G2i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2H3M7r3!0J5N6q4)9J5c8X3y4G2L8Y4c8S2j5%4c8Q4x3X3g2S2M7%4l9`. for assistance.",LF,LF,"Error: CBX-3"`

`1ED4EF10`|.`/74 10`         `je`      `short` `1ED4EF22`                  //eb

`1ED4EF12`|.`|8B15 000A0F1F` `mov`     `edx`, `dword` `ptr` `[1F0F0A00]`

`1ED4EF18`|.`|52`            `push`    `edx`

`1ED4EF19`|.`|6A 00`         `push`    `0`

`1ED4EF1B`|.`|68 38A4021F`   `push`    `1F02A438`                         `;  ASCII "Visual Assist X is unable to start your trial. You may need to reboot your system. If the problem persists, contact us at dd5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6Z5L8$3I4W2N6r3!0E0j5i4c8G2i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2H3M7r3!0J5N6q4)9J5c8X3y4G2L8Y4c8S2j5%4c8Q4x3X3g2S2M7%4l9`. for assistance.",LF,LF,"Error: CFX-3"`

`1ED4EF20`|.^|`EB A8`         `jmp`     `short` `1ED4EECA`

`1ED4EF22`|>`\8D4424 38`     `lea`     `eax`, `dword` `ptr` `[[esp]+38]`

`1ED4EF26`|.`68 70A6021F`   `push`    `1F02A670`                         `;  ASCII "EXPIRED"`

`1ED4EF4D`|.`/74 18`         `je`      `short` `1ED4EF67`/////////eb

`1ED4EF4F`|.`|8B15 000A0F1F` `mov`     `edx`, `dword` `ptr` `[1F0F0A00]`

`1ED4EF55`|.`|52`            `push`    `edx`

`1ED4EF56`|.`|6A 00`         `push`    `0`

`1ED4EF58`|.`|68 98A3021F`   `push`    `1F02A398`                         `;  ASCII "There is a problem with your license for Visual Assist X. Please contact us at 8c5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6Z5L8$3I4W2N6r3!0E0j5i4c8G2i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2H3M7r3!0J5N6q4)9J5c8X3y4G2L8Y4c8S2j5%4c8Q4x3X3g2S2M7%4l9`. for assistance.",LF,"Error: CEX-3"`

`1ED4EF5D`|.^|`E9 68FFFFFF`   `jmp`     `1ED4EECA`

`1ED4EF62`|>`|E8 199B0300`   `call`    `1ED88A80`

`1ED4EF67`|>`\833D 14080F1F>` `cmp`     `dword` `ptr` `[1F0F0814]`, 0

`1ED4EF6E`|.`74 0D`         `je`      `short` `1ED4EF7D`  ////////////////eb

`1ED4EF70`|.`68 78A3021F`   `push`    `1F02A378`                         `;  ASCII "InitInstance EdDll loaded"`

找了N多字符串，比如最核心的Patch是先找这个字符串

Visual Assist X is loaded but dormant. You should uninstall the

找到以后看到上面那个最近的Call，进去以后把这些ASC II码粘上

```
33 C0 40 C3
```

实际上就是

```
1EEBFFD0    33C0            xor     eax, eax

  1EEBFFD2    40              inc     eax

  1EEBFFD3    C3              retn
```

其他的就是改一些JMP，那些字符串搜到需要改的地址你也就晓得了……

最后一种还可以用SetEnvironmentVariable

因为我在程序里发现了这么一段
