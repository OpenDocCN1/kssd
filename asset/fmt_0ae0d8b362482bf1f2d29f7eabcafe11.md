
## 反汇编代码分析：入口点相关

`:004D6A94 E80BF4F5FF` `call 00435EA4`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:004D6976`(U), `:004D69D8`(U), `:004D6A8D`(U)

`:004D6A99 33C0` `xor eax, eax`
`:004D6A9B 5A` `pop edx`
`:004D6A9C 59` `pop ecx`
`:004D6A9D 59` `pop ecx`
`:004D6A9E 648910` `mov dword ptr fs:[eax], edx`
`:004D6AA1 68C36A4D00` `push 004D6AC3`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004D6AC1`(U)

`:004D6AA6 8D45EC` `lea eax, dword ptr [ebp-14]`
`:004D6AA9 BA04000000` `mov edx, 00000004`
`:004D6AAE E8E5DFF2FF` `call 00404A98`
`:004D6AB3 8D45FC` `lea eax, dword ptr [ebp-04]`
`:004D6AB6 E8B9DFF2FF` `call 00404A74`
`:004D6ABB C3` `ret`

---

## 进入算法 CALL：`4D69F7` `call 004C9384`

> \* Referenced by a CALL at Addresses: `:004C9BA6`, `:004D69F7`, `:004D6A4C`

`:004C9384 55` `push ebp`
`:004C9385 8BEC` `mov ebp, esp`
`:004C9387 6A00` `push 00000000`
`:004C9389 53` `push ebx`
`:004C938A 56` `push esi`
`:004C938B 8BF0` `mov esi, eax`
`:004C938D 33C0` `xor eax, eax`
`:004C938F 55` `push ebp`
`:004C9390 6852944C00` `push 004C9452`
`:004C9395 64FF30` `push dword ptr fs:[eax]`
`:004C9398 648920` `mov dword ptr fs:[eax], esp`
`:004C939B BBD9D10E00` `mov ebx, 000ED1D9`
> `====>` `EBX`= `000ED1D9`  呵呵，971225，97 年的圣诞？

`:004C93A0 8D45FC` `lea eax, dword ptr [ebp-04]`

> \* Possible StringData Ref from Data Obj -> `"wangshuang"`
> `====>` 呵呵，作者的朋友？

`:004C93A3 BA68944C00` `mov edx, 004C9468`
> `====>` `EDX`= `wangshuang`

`:004C93A8 E85FB7F3FF` `call 00404B0C`
`:004C93AD 8B45FC` `mov eax, dword ptr [ebp-04]`
`:004C93B0 E877B9F3FF` `call 00404D2C`
> `====>` 取 `wangshuang` 的长度

`:004C93B5 85C0` `test eax, eax`
> `====>` `EAX`= `A`

`:004C93B7 7E1B` `jle 004C93D4`
`:004C93B9 BA01000000` `mov edx, 00000001`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C93D2`(C)

`:004C93BE 8B4DFC` `mov ecx, dword ptr [ebp-04]`
`:004C93C1 0FB64C11FF` `movzx ecx, byte ptr [ecx+edx-01]`
> `====>` 依次取 `wangshuang` 字符的 HEX 值
> 1、 `====>` `ECX`= `77`
> …… …… 省 略 …… ……
> 10、 `====>` `ECX`= `67`

`:004C93C6 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `77` \* `5`= `00000253`
> …… …… 省 略 …… ……
> 10、 `====>` `ECX`= `67` \* `5`= `00000203`

`:004C93C9 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `00000253` \* `5`= `00000B9F`
> …… …… 省 略 …… ……
> 10、 `====>` `ECX`= `00000203` \* `5`= `00000A0F`

`:004C93CC 03D9` `add ebx, ecx`
> 1、 `====>` `EBX`= `000ED1D9` + `00000B9F`= `000EDD78`
> …… …… 省 略 …… ……
> 10、 `====>` `EBX`= `20C950EE` + `00000A0F`= `20C95AFD`

`:004C93CE 03DE` `add ebx, esi`
> 1、 `====>` `EBX`= `000EDD78` + `03A2E721`= `03B1C499`
> …… …… 省 略 …… ……
> 10、 `====>` `EBX`= `20C95AFD` + `03A2E721`= `246C421E`

`:004C93D0 42` `inc edx`
`:004C93D1 48` `dec eax`
`:004C93D2 75EA` `jne 004C93BE`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C93B7`(C)

`:004C93D4 8D45FC` `lea eax, dword ptr [ebp-04]`

> \* Possible StringData Ref from Data Obj -> `"yaoyuan"`
> `====>` 呵呵，作者的大名

`:004C93D7 BA7C944C00` `mov edx, 004C947C`
> `====>` `EDX`= `yaoyuan`

`:004C93DC E82BB7F3FF` `call 00404B0C`
`:004C93E1 8B45FC` `mov eax, dword ptr [ebp-04]`
`:004C93E4 E843B9F3FF` `call 00404D2C`
> `====>` 取 `yaoyuan` 的长度

`:004C93E9 85C0` `test eax, eax`
> `====>` `EAX`= `7`

`:004C93EB 7E1B` `jle 004C9408`
`:004C93ED BA01000000` `mov edx, 00000001`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C9406`(C)

`:004C93F2 8B4DFC` `mov ecx, dword ptr [ebp-04]`
`:004C93F5 0FB64C11FF` `movzx ecx, byte ptr [ecx+edx-01]`
> `====>` 依次取 `yaoyuan` 字符的 HEX 值
> 1、 `====>` `ECX`= `79`
> …… …… 省 略 …… ……
> 7、 `====>` `ECX`= `6E`

`:004C93FA 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `79` \* `5`= `0000025D`
> …… …… 省 略 …… ……
> 7、 `====>` `ECX`= `6E` \* `5`= `00000226`

`:004C93FD 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `0000025D` \* `5`= `00000BD1`
> …… …… 省 略 …… ……
> 7、 `====>` `ECX`= `00000226` \* `5`= `00000ABE`

`:004C9400 03D9` `add ebx, ecx`
> 1、 `====>` `EBX`= `246C421E` + `00000203`= `246C4DEF`
> …… …… 省 略 …… ……
> 7、 `====>` `EBX`= `3A3DEDBC` + `00000ABE`= `20C95AFD`

`:004C9402 03DE` `add ebx, esi`
> 1、 `====>` `EBX`= `246C4DEF` + `03A2E721`= `280F3510`
> …… …… 省 略 …… ……
> 7、 `====>` `EBX`= `3A3DF87A` + `03A2E721`= `3DE0DF9B`

`:004C9404 42` `inc edx`
`:004C9405 48` `dec eax`
`:004C9406 75EA` `jne 004C93F2`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C93EB`(C)

`:004C9408 8D45FC` `lea eax, dword ptr [ebp-04]`

> \* Possible StringData Ref from Data Obj -> `"JSBuilder"`
> `====>` 呵呵，软件的大名

`:004C940B BA8C944C00` `mov edx, 004C948C`
> `====>` `EDX`= `JSBuilder`

`:004C9410 E8F7B6F3FF` `call 00404B0C`
`:004C9415 8B45FC` `mov eax, dword ptr [ebp-04]`
`:004C9418 E80FB9F3FF` `call 00404D2C`
> `====>` 取 `JSBuilder` 的长度

`:004C941D 85C0` `test eax, eax`
> `====>` `EAX`= `9`

`:004C941F 7E1B` `jle 004C943C`
`:004C9421 BA01000000` `mov edx, 00000001`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C943A`(C)

`:004C9426 8B4DFC` `mov ecx, dword ptr [ebp-04]`
`:004C9429 0FB64C11FF` `movzx ecx, byte ptr [ecx+edx-01]`
> `====>` 依次取 `JSBuilder` 字符的 HEX 值
> 1、 `====>` `ECX`= `4A`
> …… …… 省 略 …… ……
> 9、 `====>` `ECX`= `72`

`:004C942E 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `4A` \* `5`= `00000172`
> …… …… 省 略 …… ……
> 9、 `====>` `ECX`= `72` \* `5`= `0000023A`
