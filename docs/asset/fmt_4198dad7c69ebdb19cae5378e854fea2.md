
:004F6AB0 `27` `daa`   
:004F6AB1 `206E6F` `and` byte ptr `[esi+6F]`, ch   
:004F6AB4 `7420` `je` 004F6AD6   
:004F6AB6 `666F` `outsw`   
:004F6AB8 `756E` `jne` 004F6B28   
:004F6ABA `64000400` `add` byte ptr `fs:[eax+eax]`, al   
:004F6ABE `0000` `add` byte ptr `[eax]`, al   
:004F6AC0 `53` `push` ebx   
:004F6AC1 `8B15E8BD4F00` `mov` edx, dword ptr `[004FBDE8]`   
:004F6AC7 `833A00` `cmp` dword ptr `[edx]`, 00000000   
:004F6ACA `740A` `je` 004F6AD6 **设断** （1）   
:004F6ACC `8B1DE8BD4F00` `mov` ebx, dword ptr `[004FBDE8]`   
:004F6AD2 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6AD4 `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
`:004F6AB4(C), :004F6ACA(C)`

:004F6AD6 `5B` `pop` ebx   
:004F6AD7 `C3` `ret`   

* Referenced by a CALL at Address:   
`:004F7BF2`

:004F6AD8 `53` `push` ebx   
:004F6AD9 `8B1528BF4F00` `mov` edx, dword ptr `[004FBF28]`   
:004F6ADF `833A00` `cmp` dword ptr `[edx]`, 00000000   
:004F6AE2 `740A` `je` 004F6AEE **设断** （2）   
:004F6AE4 `8B1D28BF4F00` `mov` ebx, dword ptr `[004FBF28]`   
:004F6AEA `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6AEC `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6AE2(C)`

:004F6AEE `5B` `pop` ebx   
:004F6AEF `C3` `ret`   

* Referenced by a CALL at Address:   
`:004F7C12`

:004F6AF0 `53` `push` ebx   
:004F6AF1 `8B1578C14F00` `mov` edx, dword ptr `[004FC178]`   
:004F6AF7 `833A00` `cmp` dword ptr `[edx]`, 00000000   
:004F6AFA `740A` `je` 004F6B06 **设断** （3）   
:004F6AFC `8B1D78C14F00` `mov` ebx, dword ptr `[004FC178]`   
:004F6B02 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6B04 `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6AFA(C)`

:004F6B06 `5B` `pop` ebx   
:004F6B07 `C3` `ret`   

* Referenced by a CALL at Address:   
`:004F7C32`

:004F6B08 `53` `push` ebx   
:004F6B09 `8B15A0C04F00` `mov` edx, dword ptr `[004FC0A0]`   
:004F6B0F `833A00` `cmp` dword ptr `[edx]`, 00000000   
:004F6B12 `740A` `je` 004F6B1E **设断** （4）   
:004F6B14 `8B1DA0C04F00` `mov` ebx, dword ptr `[004FC0A0]`   
:004F6B1A `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6B1C `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6B12(C)`

:004F6B1E `5B` `pop` ebx   
:004F6B1F `C3` `ret`   

* Referenced by a CALL at Address:   
`:004F4F6E`

:004F6B20 `53` `push` ebx   
:004F6B21 `8B157CBC4F00` `mov` edx, dword ptr `[004FBC7C]`   
:004F6B27 `833A00` `cmp` dword ptr `[edx]`, 00000000   
:004F6B2A `740A` `je` 004F6B36 **设断** （5）   
:004F6B2C `8B1D7CBC4F00` `mov` ebx, dword ptr `[004FBC7C]`   
:004F6B32 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6B34 `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6B2A(C)`

:004F6B36 `5B` `pop` ebx   
:004F6B37 `C3` `ret`   
:004F6B38 `53` `push` ebx   
:004F6B39 `8B15E4BD4F00` `mov` edx, dword ptr `[004FBDE4]`   
:004F6B3F `833A00` `cmp` dword ptr `[edx]`, 00000000   
:004F6B42 `740A` `je` 004F6B4E **设断** （6）   
:004F6B44 `8B1DE4BD4F00` `mov` ebx, dword ptr `[004FBDE4]`   
:004F6B4A `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6B4C `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6B42(C)`

:004F6B4E `5B` `pop` ebx   
:004F6B4F `C3` `ret`   

* Referenced by a CALL at Address:   
`:004F7CC2`

:004F6B50 `53` `push` ebx   
:004F6B51 `A150BD4F00` `mov` eax, dword ptr `[004FBD50]`   
:004F6B56 `833800` `cmp` dword ptr `[eax]`, 00000000   
:004F6B59 `740C` `je` 004F6B67 **设断** （7）   
:004F6B5B `8B1D50BD4F00` `mov` ebx, dword ptr `[004FBD50]`   
:004F6B61 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6B63 `8BC2` `mov` eax, edx   
:004F6B65 `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6B59(C)`

:004F6B67 `5B` `pop` ebx   
:004F6B68 `C3` `ret`   
:004F6B69 `8D4000` `lea` eax, dword ptr `[eax+00]`   
:004F6B6C `55` `push` ebp   
:004F6B6D `8BEC` `mov` ebp, esp   
:004F6B6F `53` `push` ebx   
:004F6B70 `A144BE4F00` `mov` eax, dword ptr `[004FBE44]`   
:004F6B75 `833800` `cmp` dword ptr `[eax]`, 00000000   
:004F6B78 `7411` `je` 004F6B8B **设断** （8）   
:004F6B7A `8B1D44BE4F00` `mov` ebx, dword ptr `[004FBE44]`   
:004F6B80 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6B82 `8BC2` `mov` eax, edx   
:004F6B84 `8BD1` `mov` edx, ecx   
:004F6B86 `8A4D08` `mov` cl, byte ptr `[ebp+08]`   
:004F6B89 `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6B78(C)`

:004F6B8B `5B` `pop` ebx   
:004F6B8C `5D` `pop` ebp   
:004F6B8D `C20400` `ret` 0004   
:004F6B90 `55` `push` ebp   
:004F6B91 `8BEC` `mov` ebp, esp   
:004F6B93 `53` `push` ebx   
:004F6B94 `A100C14F00` `mov` eax, dword ptr `[004FC100]`   
:004F6B99 `833800` `cmp` dword ptr `[eax]`, 00000000   
:004F6B9C `7411` `je` 004F6BAF **设断** （9）   
:004F6B9E `8B1D00C14F00` `mov` ebx, dword ptr `[004FC100]`   
:004F6BA4 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6BA6 `8BC2` `mov` eax, edx   
:004F6BA8 `8BD1` `mov` edx, ecx   
:004F6BAA `8A4D08` `mov` cl, byte ptr `[ebp+08]`   
:004F6BAD `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6B9C(C)`

:004F6BAF `5B` `pop` ebx   
:004F6BB0 `5D` `pop` ebp   
:004F6BB1 `C20400` `ret` 0004   
:004F6BB4 `55` `push` ebp   
:004F6BB5 `8BEC` `mov` ebp, esp   
:004F6BB7 `53` `push` ebx   
:004F6BB8 `A170BD4F00` `mov` eax, dword ptr `[004FBD70]`   
:004F6BBD `833800` `cmp` dword ptr `[eax]`, 00000000   
:004F6BC0 `7411` `je` 004F6BD3 **设断** （10）   
:004F6BC2 `8B1D70BD4F00` `mov` ebx, dword ptr `[004FBD70]`   
:004F6BC8 `8B1B` `mov` ebx, dword ptr `[ebx]`   
:004F6BCA `8BC2` `mov` eax, edx   
:004F6BCC `8BD1` `mov` edx, ecx   
:004F6BCE `8A4D08` `mov` cl, byte ptr `[ebp+08]`   
:004F6BD1 `FFD3` `call` ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
`:004F6BC0(C)`

:004F6BD3 `5B` `pop` ebx
