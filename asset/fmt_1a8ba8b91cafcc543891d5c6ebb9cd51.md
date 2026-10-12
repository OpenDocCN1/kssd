
:1000111B `8D0C80`               `lea ecx, dword ptr [eax+4*eax]`   //ECX=5*EAX。

:1000111E `8D1489`               `lea edx, dword ptr [ecx+4*ecx]`   //EDX=5*ECX。

:10001121 `89542410`             `mov dword ptr [esp+10], edx`      //保存 EDX 在第一组后面。

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:10001148(C)`

:10001125 `8D7C1C10`             `lea edi, dword ptr [esp+ebx+10]` //第二段真码地址送 EDI。

:10001129 `8A07`                 `mov al, byte ptr [edi]`          //取每个真码字符。

:1000112B `50`                   `push eax`

:1000112C `53`                   `push ebx`

:1000112D `43`                   `inc ebx`

:1000112E `E83D010000`           `call 10001270`                   //变换字符。     （*****）

:10001133 `83C408`               `add esp, 00000008`

:10001136 `33C9`                 `xor ecx, ecx`

:10001138 `8AC8`                 `mov cl, al`

:1000113A `51`                   `push ecx`

:1000113B `E810040000`           `call 10001550`                   // 把字符变换为 1-9 或者 a-z 之间。

:10001140 `83C404`               `add esp, 00000004`

:10001143 `8807`                 `mov byte ptr [edi], al`

:10001145 `83FB04`               `cmp ebx, 00000004`

:10001148 `7CDB`                 `jl 10001125`

:1000114A `33C9`                 `xor ecx, ecx`

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:1000116F(C)`

:1000114C `0FBE440E05`           `movsx eax, byte ptr [esi+ecx+05]` //下面比较第二段码。

:10001151 `83F841`               `cmp eax, 00000041`

:10001154 `7C08`                 `jl 1000115E`

:10001156 `83F85A`               `cmp eax, 0000005A`

:10001159 `7F03`                 `jg 1000115E`

:1000115B `83C020`               `add eax, 00000020`

*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:10001154(C), :10001159(C)`

:1000115E `0FBE540C10`           `movsx edx, byte ptr [esp+ecx+10]`

:10001163 `3BD0`                 `cmp edx, eax`

:10001165 `0F85D6000000`         `jne 10001241`

:1000116B `41`                   `inc ecx`

:1000116C `83F904`               `cmp ecx, 00000004`

:1000116F `7CDB`                 `jl 1000114C`

:10001171 `8B442410`             `mov eax, dword ptr [esp+10]` //EAX 保存第二组码。

:10001175 `8B4C2410`             `mov ecx, dword ptr [esp+10]` //ECX 保存第二组码。

:10001179 `3344240C`             `xor eax, dword ptr [esp+0C]` //EAX 与第一组异或。

:1000117D `33DB`                 `xor ebx, ebx`

:1000117F `83C009`               `add eax, 00000009`             //EAX 加 9。

:10001182 `0FAF44240C`           `imul eax, dword ptr [esp+0C]` //EAX 乘以第一组码。

:10001187 `8D1488`               `lea edx, dword ptr [eax+4*ecx]` //EDX=4*ECX+EAX。

:1000118A `89542414`             `mov dword ptr [esp+14], edx`    //保存结果。

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:100011A4(C)`

:1000118E `8D7C1C14`             `lea edi, dword ptr [esp+ebx+14]`   //取真码地址。

:10001192 `43`                   `inc ebx`

:10001193 `0FBE07`               `movsx eax, byte ptr [edi]`         //取每位真码送 EAX。

:10001196 `50`                   `push eax`

:10001197 `E8B4030000`           `call 10001550`                     // 把字符变换为 1-9 或者 a-z 之间。

:1000119C `83C404`               `add esp, 00000004`

:1000119F `8807`                 `mov byte ptr [edi], al`

:100011A1 `83FB04`               `cmp ebx, 00000004`

:100011A4 `7CE8`                 `jl 1000118E`

:100011A6 `33C9`                 `xor ecx, ecx`

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:100011CB(C)`

:100011A8 `0FBE440E0A`           `movsx eax, byte ptr [esi+ecx+0A]`  //下面比较第三段注册码。

:100011AD `83F841`               `cmp eax, 00000041`

:100011B0 `7C08`                 `jl 100011BA`

:100011B2 `83F85A`               `cmp eax, 0000005A`

:100011B5 `7F03`                 `jg 100011BA`

:100011B7 `83C020`               `add eax, 00000020`

*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:100011B0(C), :100011B5(C)`

:100011BA `0FBE540C14`           `movsx edx, byte ptr [esp+ecx+14]`

:100011BF `3BD0`                 `cmp edx, eax`

:100011C1 `0F8585000000`         `jne 1000124C`

:100011C7 `41`                   `inc ecx`

:100011C8 `83F904`               `cmp ecx, 00000004`

:100011CB `7CDB`                 `jl 100011A8`

:100011CD `8B4C2410`             `mov ecx, dword ptr [esp+10]`  //ECX 保存第二段注册码。

:100011D1 `8B442414`             `mov eax, dword ptr [esp+14]`  //EAX 保存第三段注册码。

:100011D5 `0FAF4C2414`           `imul ecx, dword ptr [esp+14]` //ECX 乘以第三段注册码。

:100011DA `83C106`               `add ecx, 00000006`             //ECX 加 6。

:100011DD `33FF`                 `xor edi, edi`

:100011DF `0FAF4C240C`           `imul ecx, dword ptr [esp+0C]` //ECX 乘以第一段注册码。

:100011E4 `2BC8`                 `sub ecx, eax`                  //ECX=ECX-EAX。

:100011E6 `8D14C1`               `lea edx, dword ptr [ecx+8*eax]` //EDX=ECX+8*EAX。

:100011E9 `89542418`             `mov dword ptr [esp+18], edx`    //保存结果。

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:10001203(C)`

:100011ED `8D5C3C18`             `lea ebx, dword ptr [esp+edi+18]`  //取真码地址送 EBX。

:100011F1 `47`                   `inc edi`

:100011F2 `0FBE03`               `movsx eax, byte ptr [ebx]`        //取每位字符送 EAX。

:100011F5 `50`                   `push eax`

:100011F6 `E855030000`           `call 10001550`                    // 把字符变换为 1-9 或者 a-z 之间。

:100011FB `83C404`               `add esp, 00000004`

:100011FE `8803`                 `mov byte ptr [ebx], al`

:10001200 `83FF04`               `cmp edi, 00000004`

:10001203 `7CE8`                 `jl 100011ED`

:10001205 `33C9`                 `xor ecx, ecx`

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:10001226(C)`

:10001207 `0FBE440E0F`           `movsx eax, byte ptr [esi+ecx+0F]` //下面比较第四段注册码。

:1000120C `83F841`               `cmp eax, 00000041`

:1000120F `7C08`                 `jl 10001219`

:10001211 `83F85A`               `cmp eax, 0000005A`

:10001214 `7F03`                 `jg 10001219`

:10001216 `83C020`               `add eax, 00000020`

*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:1000120F(C), :10001214(C)`

:10001219 `0FBE540C18`           `movsx edx, byte ptr [esp+ecx+18]`

:1000121E `3BD0`                 `cmp edx, eax`

:10001220 `7535`                 `jne 10001257`

:10001222 `41`                   `inc ecx`

:10001223 `83F904`               `cmp ecx, 00000004`

:10001226 `7CDF`                 `jl 10001207`

:10001228 `B801000000`           `mov eax, 00000001`       //注册码正确则置 EAX 为 1。

:1000122D `5F`                   `pop edi`

:1000122E `5E`                   `pop esi`

:1000122F `5B`                   `pop ebx`

:10001230 `83C420`               `add esp, 00000020`

:10001233 `C20800`               `ret 0008`

\================================================================

下面是 `100010BA` 处 `call 10001290`（**）函数的代码：

:10001290 `81EC00020000`           `sub esp, 00000200`

:10001296 `33C0`                 `xor eax, eax`

:10001298 `B980000000`           `mov ecx, 00000080`

:1000129D `53`                   `push ebx`

:1000129E `56`                   `push esi`

:1000129F `33DB`                 `xor ebx, ebx`

:100012A1 `57`                   `push edi`

:100012A2 `55`                   `push ebp`

:100012A3 `8D7C2410`             `lea edi, dword ptr [esp+10]`

:100012A7 `F3`                   `repz`

:100012A8 `AB`                   `stosd`

:100012A9 `8BB42414020000`       `mov esi, dword ptr [esp+00000214]` //用户名地址送 ESI。

:100012B0 `56`                   `push esi`

*   Reference To: `KERNEL32.lstrlenA`, Ord:0308h

:100012B1 `FF1504600010`           `Call dword ptr [10006004]`        //取用户名长度。
