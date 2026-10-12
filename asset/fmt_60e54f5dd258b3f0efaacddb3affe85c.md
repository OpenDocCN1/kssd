
> `:005071FA BA02000000` `mov edx, 00000002`

> `:005071FF E80C23F0FF` `call 00409510`

> `:00507204 EB17` `jmp 0050721D`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:005070F4(C), :0050717B(C)

> \* Possible StringData Ref from Code Obj -> `"认证码及序列号错误！"`

> `:00507206 B840735000` `mov eax, 00507340`

> `:0050720B E8EC31F3FF` `call 0043A3FC`

> `:00507210 33D2` `xor edx, edx`

> `:00507212 8B8350040000` `mov eax, dword ptr [ebx+00000450]`

> `:00507218 E81BA0F3FF` `call 00441238`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:005070D7(U), :00507204(U)

> `:0050721D 33C0` `xor eax, eax`

> `:0050721F 5A` `pop edx`

> `:00507220 59` `pop ecx`

> `:00507221 59` `pop ecx`

> `:00507222 648910` `mov dword ptr fs:[eax], edx`

> `:00507225 6875725000` `push 00507275`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00507273(U)

> `:0050722A 8D8558FFFFFF` `lea eax, dword ptr [ebp+FFFFFF58]`

> `:00507230 BA02000000` `mov edx, 00000002`

> `:00507235 E8AED5EFFF` `call 004047E8`

> `:0050723A 8D8560FFFFFF` `lea eax, dword ptr [ebp+FFFFFF60]`

> `:00507240 E87FD5EFFF` `call 004047C4`

> `:00507245 8D8564FFFFFF` `lea eax, dword ptr [ebp+FFFFFF64]`

> `:0050724B BA03000000` `mov edx, 00000003`

> `:00507250 E893D5EFFF` `call 004047E8`

> `:00507255 8D8570FFFFFF` `lea eax, dword ptr [ebp+FFFFFF70]`

> `:0050725B BA02000000` `mov edx, 00000002`

> `:00507260 E883D5EFFF` `call 004047E8`

> `:00507265 8D45FC` `lea eax, dword ptr [ebp-04]`

> `:00507268 E857D5EFFF` `call 004047C4`

> `:0050726D C3` `ret`

>
> `4dd466d0e1e0cc123afbe504da2462ca`
