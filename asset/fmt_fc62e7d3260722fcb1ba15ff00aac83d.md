
`0046 00BB4153:014A1AEA rep movsb [register],[esp] (01 bytes) ; mov eax,ecx`

`0047 00BB4161:0148771E mov register,context.esp`

`0048 00BB416F:0150C5DD push dword ptr [register]`

`0049 00BB417D:0148771E mov register,addr_of_context.eax`

`0050 00BB418B:014A1AEA rep movsb [register],[esp] (04 bytes) ; mov eax,[esp]`

`0051 00BB4199:0148771E mov register,context.esp`

`0052 00BB41A7:0152C47F ; 未实现,可以猜测是 add esp,4`

`0053 00BB41B5:0150C5DD push register`

`0054 00BB41C3:014A1AEA rep movsb context.esp,[esp] (04 bytes)`

`0055 00BB41D1:01450073 mov register,0BA18350`

`0056 00BB41DF:0148771E mov register,addr_of_context.ebp`

`0057 00BB41ED:0150C5DD push dword ptr [register]`

`0058 00BB41FB:0150C5DD push 00000005`

`0059 00BB4209:014F7A55 ; 未实现`

`0060 00BB4217:014A1AEA rep movsb [register],[esp] (04 bytes)`

`0061 00BB4225:01461D1F`

`0062 00BB4233:0148771E mov register,context.esp`

`0063 00BB4241:0150C5DD push dword ptr [register]`

`0064 00BB424F:0148771E mov register,addr_of_context.ebp`

`0065 00BB425D:014A1AEA rep movsb [register],[esp] (04 bytes)`

`0066 00BB426B:0148771E mov register,context.esp ; 可以看到 mov ebp,esp 在这里`

结论是，被转换为`pcode`方式运行的代码，事先被混淆过了。再用这种手工式的修复方法，已经不可行了。

1行真正的汇编指令，大约对应10行`pcode`代码，每条`pcode`执行时，对应的汇编指令一般超出2000行，真正有点疯狂呵呵。
