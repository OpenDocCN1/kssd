
`004D59A3`   |.  `8D7D F0`                  `lea edi,dword ptr ss:[ebp-10]`  ;变形 MD5 计算的结果  

\---------------------------------------------------------------------  

`004D596C`   |.  `33C0`                    `xor eax,eax`  

`004D596E`   |.  `5A`                      `pop edx`  

`004D596F`   |.  `59`                      `pop ecx`  

`004D5970`   |.  `59`                      `pop ecx`  

`004D5971`   |.  `64:8910`                 `mov dword ptr fs:[eax],edx`  

`004D5974`   |.  `68 89594D00`             `push unpack.004D5989`  

`004D5979`   |>  `8D45 FC`                 `lea eax,dword ptr ss:[ebp-4]`  

`004D597C`   |.  `E8 C3E7F2FF`             `call unpack.00404144`  

`004D5981`   \\.  `C3`                      `retn`  

\--------------------------------------------------------------------------------------  

`00539AD2`   |.  `8B45 D0`                 `mov eax,dword ptr ss:[ebp-30]`  

`00539AD5`   |.  `8D4D D4`                 `lea ecx,dword ptr ss:[ebp-2C]`  

`00539AD8`   |.  `BA 0A000000`             `mov edx,0A`                  ;取 `MD5(name)` 的前 10 个字符  

`00539ADD`   <>|.  `E8 52DCEFFF`           `call unpack.00437734`        ;  ->`StrUtils.LeftStr(AnsiString;Integer):AnsiString;overload;`  

`00539AE2`   |.  `8B55 D4`                 `mov edx,dword ptr ss:[ebp-2C]`        ;两个联起来  

`00539AE5`   |.  `58`                      `pop eax`  

`00539AE6`   <>|.  `E8 21A9ECFF`           `call unpack.0040440C`        ;  ->`System.@LStrCat;`  

`00539AEB`   |.  `8B45 DC`                 `mov eax,dword ptr ss:[ebp-24]`  

`00539AEE`   |.  `8D55 E0`                 `lea edx,dword ptr ss:[ebp-20]`  

`00539AF1`   |.  `E8 5AE2FFFF`             `call unpack.00537D50`  

`00539AF6`   |.  `8B55 E0`                 `mov edx,dword ptr ss:[ebp-20]`  

`00539AF9`   |.  `58`                      `pop eax`  

`00539AFA`   <>|.  `E8 51AAECFF`           `call unpack.00404550`        ;  ->`System.@LStrCmp;`  

对于验证，是把假码经过一系列的浮点运算，得到一个数，然后把真码也经过同样的浮点运算得到一个数，两个进行比较，若相等则注册成功

兄弟们可以做注册机了，用标准的 `DES` 算法，密钥为 `"Security"`，对其进行加密，取前 10 个字符，然后再用提到的变形 MD5 算法（只需要改一下四个常数即可），也取前 10 个字符，两个联接起来就是正确的 `sn`

`4dd466d0e1e0cc123afbe504da2462ca`
