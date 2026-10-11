
:10001125 `0FAFD0` `imul edx, eax` //EDX=EDX\*EAX。  

:10001128 `8954240C` `mov dword ptr [esp+0C], edx` //结果送到[ESP+C]处。  

:1000112C `33F6` `xor esi, esi`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :1000114F(C)  

|  

:1000112E `8A44340C` `mov al, byte ptr [esp+esi+0C]`  

:10001132 `50` `push eax`  

:10001133 `56` `push esi`  

:10001134 `E847030000` `call 10001480` //这个函数把变换得到的新 4 位注册码进一步变换。（\*\*\*\*）  

:10001139 `25FF000000` `and eax, 000000FF`  

:1000113E `50` `push eax`  

:1000113F `E8AC010000` `call 100012F0` //变为可见字符。  

:10001144 `83C40C` `add esp, 0000000C`  

:10001147 `8844340C` `mov byte ptr [esp+esi+0C], al`  

:1000114B `46` `inc esi`  

:1000114C `83FE04` `cmp esi, 00000004`  

:1000114F `7CDD` `jl 1000112E`  

:10001151 `33C9` `xor ecx, ecx`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :10001176(C)  

|  

:10001153 `0FBE440F05` `movsx eax, byte ptr [edi+ecx+05]` //下面是比较真假注册码的循环。  

:10001158 `83F841` `cmp eax, 00000041`  

:1000115B `7C08` `jl 10001165`  

:1000115D `83F85A` `cmp eax, 0000005A`  

:10001160 `7F03` `jg 10001165`  

:10001162 `83C020` `add eax, 00000020`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :1000115B(C), :10001160(C)  

|  

:10001165 `0FBE540C0C` `movsx edx, byte ptr [esp+ecx+0C]`  

:1000116A `3BC2` `cmp eax, edx`  

:1000116C `0F85BA000000` `jne 1000122C`  

:10001172 `41` `inc ecx`  

:10001173 `83F904` `cmp ecx, 00000004`  

:10001176 `7CDB` `jl 10001153`  

:10001178 `8B44240C` `mov eax, dword ptr [esp+0C]` //第一段注册码送 EAX。  

:1000117C `8B4C2408` `mov ecx, dword ptr [esp+08]` //第二段注册码送 ECX。  

:10001180 `8BD0` `mov edx, eax`  

:10001182 `33D1` `xor edx, ecx` //异或运算  

:10001184 `42` `inc edx` //加 1。  

:10001185 `0FAFD1` `imul edx, ecx` //EDX=EDX\*ECX  

:10001188 `03D0` `add edx, eax` //EDX=EDX+EAX。  

:1000118A `33F6` `xor esi, esi`  

:1000118C `89542410` `mov dword ptr [esp+10], edx` //保存结果。  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :100011A6(C)  

|  

:10001190 `0FBE443410` `movsx eax, byte ptr [esp+esi+10]`  

:10001195 `50` `push eax`  

:10001196 `E855010000` `call 100012F0` //处理刚刚得到的 4 位结果。  

:1000119B `83C404` `add esp, 00000004`  

:1000119E `88443410` `mov byte ptr [esp+esi+10], al` //保存  

:100011A2 `46` `inc esi`  

:100011A3 `83FE04` `cmp esi, 00000004`  

:100011A6 `7CE8` `jl 10001190`  

:100011A8 `33C9` `xor ecx, ecx`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :100011C9(C)  

|  

:100011AA `0FBE440F0A` `movsx eax, byte ptr [edi+ecx+0A]` //下面是比较第三段注册码的循环。  

:100011AF `83F841` `cmp eax, 00000041`  

:100011B2 `7C08` `jl 100011BC`  

:100011B4 `83F85A` `cmp eax, 0000005A`  

:100011B7 `7F03` `jg 100011BC`  

:100011B9 `83C020` `add eax, 00000020`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :100011B2(C), :100011B7(C)  

|  

:100011BC `0FBE540C10` `movsx edx, byte ptr [esp+ecx+10]`  

:100011C1 `3BC2` `cmp eax, edx`  

:100011C3 `7567` `jne 1000122C`  

:100011C5 `41` `inc ecx`  

:100011C6 `83F904` `cmp ecx, 00000004`  

:100011C9 `7CDF` `jl 100011AA`  

:100011CB `8B4C240C` `mov ecx, dword ptr [esp+0C]` //第二段注册码送 ECX。  

:100011CF `8B442408` `mov eax, dword ptr [esp+08]` //第一段注册码送 EAX。  

:100011D3 `0FAFC8` `imul ecx, eax` //相乘  

:100011D6 `41` `inc ecx` //ECX 增一。  

:100011D7 `0FAF4C2410` `imul ecx, dword ptr [esp+10]` //ECX 与第三段相乘。  

:100011DC `03C8` `add ecx, eax` //ECX 加 EAX。  

:100011DE `33F6` `xor esi, esi`  

:100011E0 `894C2414` `mov dword ptr [esp+14], ecx` //保存。  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :100011FA(C)  

|  

:100011E4 `0FBE543414` `movsx edx, byte ptr [esp+esi+14]` //处理刚刚得到的第四段注册码。  

:100011E9 `52` `push edx`  

:100011EA `E801010000` `call 100012F0`  

:100011EF `83C404` `add esp, 00000004`  

:100011F2 `88443414` `mov byte ptr [esp+esi+14], al`  

:100011F6 `46` `inc esi`  

:100011F7 `83FE04` `cmp esi, 00000004`  

:100011FA `7CE8` `jl 100011E4`  

:100011FC `33C9` `xor ecx, ecx`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :1000121D(C)  

|  

:100011FE `0FBE440F0F` `movsx eax, byte ptr [edi+ecx+0F]` //下面是第四段注册码的比较循环。  

:10001203 `83F841` `cmp eax, 00000041`  

:10001206 `7C08` `jl 10001210`  

:10001208 `83F85A` `cmp eax, 0000005A`  

:1000120B `7F03` `jg 10001210`  

:1000120D `83C020` `add eax, 00000020`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :10001206(C), :1000120B(C)  

|  

:10001210 `0FBE540C14` `movsx edx, byte ptr [esp+ecx+14]`  

:10001215 `3BC2` `cmp eax, edx`  

:10001217 `7513` `jne 1000122C`  

:10001219 `41` `inc ecx`  

:1000121A `83F904` `cmp ecx, 00000004`  

:1000121D `7CDF` `jl 100011FE`  

:1000121F `5F` `pop edi`  

:10001220 `B801000000` `mov eax, 00000001` //注册码正确则让 EAX=1。  

:10001225 `5E` `pop esi`  

:10001226 `83C420` `add esp, 00000020`  

:10001229 `C20800` `ret 0008`  

======================================================================  

下面分析 :100010B7 处的函数 `call 10001240`：  

| :100010B7  

|  

:10001240 `81EC00020000` `sub esp, 00000200`  

:10001246 `B980000000` `mov ecx, 00000080`  

:1000124B `33C0` `xor eax, eax`  

:1000124D `53` `push ebx`  

:1000124E `55` `push ebp`  

:1000124F `56` `push esi`  

:10001250 `8BB42410020000` `mov esi, dword ptr [esp+00000210]`  

:10001257 `57` `push edi`  

:10001258 `8D7C2410` `lea edi, dword ptr [esp+10]`  

:1000125C `F3` `repz`  

:1000125D `AB` `stosd`  

:1000125E `56` `push esi`  

:1000125F `33ED` `xor ebp, ebp`  

\* Reference To: `KERNEL32.lstrlenA`, Ord:0308h  

|  

:10001261 `FF1504600010` `Call dword ptr [10006004]` //求串长。  

:10001267 `8BD8` `mov ebx, eax`  

:10001269 `81FB00020000` `cmp ebx, 00000200`  

:1000126F `7612` `jbe 10001283`  

:10001271 `B980000000` `mov ecx, 00000080`  

:10001276 `8D7C2410` `lea edi, dword ptr [esp+10]`  

:1000127A `BB00020000` `mov ebx, 00000200`  

:1000127F `F3` `repz`  

:10001280 `A5` `movsd`  

:10001281 `EB0C` `jmp 1000128F`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :1000126F(C)  

|  
