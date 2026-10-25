
\* Reference to control `TZ_GgkmFORM.ComboBox1` : `TComboBox`

`004FFCD4`   `8B83D8020000`           `mov` 
`eax`, [`ebx+$02D8`]

`004FFCDA`   `E895F4F2FF`             `call`
`0042F174`

7、单击`Xznlunit`,双击右面的`FORMShow`,

`0051EB6E`   `80B80403000000`         `cmp`
`byte ptr [eax+$0304]`, `$00`

`0051EB75`   `7579`
  `jnz`     `0051EBF0` ⑧*****

`0051EB77`   `33D2`
  `xor`     `edx`, `edx`

\* Reference to control `TZ_XznlFORM.Button4` : `TButton`

`0051EB79`   `8B83E8020000`           `mov`
`eax`, [`ebx+$02E8`]

`0051EB7F`   `8B08`
  `mov`     `ecx`, [`eax`]

\* Possible reference to virtual method `TButton.OFFS_5C`

`0051EB81`   `FF515C`
`call`    `dword ptr [ecx+$5C]`

`0051EB84`   `33D2`
  `xor`     `edx`, `edx`

6、关闭此文件。

7、再用十六进制的编辑器如`Hex workshop3.1`编辑此文件。

查找`ASCⅡ`码

①`80B804030000007524A194FE5300E8C670F1FF`改`75`为`74`

②`75 24 A1 94 FE 53 00 E8 A2 68 F1 FF A1 EC DF 53 00`　　改`75`为`74`

③`75 24 A1 94 FE 53 00 E8 26 68 F1 FA 1B 8E 45 53 00`　　改`75`为`74`

④`75 10 BA E4 39 53 00 A1 94 FE 53 00 E8 A9 B7 EFFF`　　改`75`为`74`

⑤`75 2E BA 60 45 03 53 00 8B 83 EC 02 00 00 E8 AE B1 EFFF`　改`75`为`74`

⑥`75 0F BA 40 06 50 00 A1 04 FE 53 00 E8 3D EB F2 FF`   改`75`为`74`

⑦`75 1D 33 D2 8B 83 D8 02 00 00`
改`75`为`74`

⑧`75 79 33 D2 8B 83 E8 02 00 08 B0 8`             改`75`为`74`

修改完成后，就变成正式版。没有了功能限制。
