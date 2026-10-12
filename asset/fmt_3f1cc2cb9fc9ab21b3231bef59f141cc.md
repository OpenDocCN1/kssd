
:0043E769 8944241C `mov dword ptr [esp+1C], eax`

:0043E76D 3BC3 `cmp eax, ebx`

:0043E76F C68424B405000004 `mov byte ptr [esp+000005B4], 04`

:0043E777 740B `je 0043E784`

:0043E779 8BC8 `mov ecx, eax`

:0043E77B E8D07B0000 `call 00446350`

:0043E780 8BF0 `mov esi, eax`

:0043E782 EB02 `jmp 0043E786`

.
.
.

这里没有稿费，所以略去中间无数过程。为节省大家时间，一直按 `F10` 代过吧，到了这里：

:0043ED89 68E8654800 `push 004865E8`

:0043ED8E 56 `push esi`

:0043ED8F E869390100 `call 004526FD`

:0043ED94 8B4C2420 `mov ecx, dword ptr [esp+20]` *取到算码后的结果，本例中是：`3AF5CDFE=989187582`（十进制的）你把 `989187582` 改为 `989187419` 试试！*

:0043ED98 BE83000000 `mov esi, 00000083` *好像很熟哦！这不是 `131` 吗？（十进制）*

:0043ED9D 8BC1 `mov eax, ecx` *结果送到 `eax` 中做除数*

:0043ED9F 83C40C `add esp, 0000000C`

:0043EDA2 99 `cdq`

:0043EDA3 F7FE `idiv esi` *除法，余数在 `edx` 中。*

:0043EDA5 85D2 `test edx, edx` *又检验是否为零。*

:0043EDA7 7433 `jne 0043EDDC` *不能整除的跳走，当然是出错了！我猜不出注册码来。只有该这儿了。改为 `je`*

:0043EDA9 B83DCB883E `mov eax, 3E88CB3D`

:0043EDAE F7E9 `imul ecx`

:0043EDB0 C1FA05 `sar edx, 05`

:0043EDB3 8BC2 `mov eax, edx`

:0043EDB5 C1E81F `shr eax, 1F`

:0043EDB8 03D0 `add edx, eax`

:0043EDBA 81FAFF000000 `cmp edx, 000000FF`

:0043EDC0 7E1A `jle 0043EDDC`

:0043EDC2 51 `push ecx`

到次成功了！在启动试试！哈哈！注册版了吧！呵呵....

不对之处请您指教！

小球[CCG]
2001/4/7 4dd466d0e1e0cc123afbe504da2462ca
