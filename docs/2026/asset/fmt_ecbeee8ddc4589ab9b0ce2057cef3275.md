
```
add    edx, [ebp+var_24]    ;sd+c1

xor    eax, edx        ;(sd<<4+ka) xor (sd+c1)

mov    ecx, [ebp+var_10]    ;sd

shr    ecx, 5
;sd>>5
add    ecx, [ebp+var_20]    ;sd>>5+kd

xor    eax, ecx
;(sd<<4+ka) xor (sd+c1) xor (sd>>5+kd)

add    [ebp+var_C], eax    ;sc=sc+[(sd<<4+ka) xor (sd+c1) xor (sd>>5+kd)]

mov    eax, [ebp+var_4]    ;sa
shl    eax, 4
;sa<<4

add    eax, [ebp+var_1C]    ;sa<<4+kc

mov    edx, [ebp+var_4]    ;sa
add    edx, [ebp+var_24]    ;sa+c1

xor    eax, edx        ;(sa<<4+kc) xor (sa+c1)

mov    ecx, [ebp+var_4]    ;sa
shr    ecx, 5
;sa>>5
add    ecx, [ebp+var_18]    ;sa>>5+kb

xor    eax, ecx
;(sa<<4+kc) xor (sa+c1) xor (sa>>5+kb)

add    [ebp+var_10], eax    ;sd=sd+[(sa<<4+kc) xor (sa+c1) xor (sa>>5+kb)]

mov    eax, [ebp+var_2C]    ;r
dec    eax
;r-1
mov    [ebp+var_2C], eax    ;r=r-1

jnz    @B

mov    eax, [ebp+arg_0]    ;pData
mov    edx, [ebp+var_4]    ;sa
mov    [eax], edx
;A=sa

mov    edx, [ebp+var_8]    ;sb
mov    [eax+4], edx
;B=sb

mov    edx, [ebp+var_C]    ;sc
mov    [eax+8], edx
;C=sc

mov    edx, [ebp+var_10]    ;sd
mov    [eax+0Ch], edx
;D=sd

mov    esp, ebp
pop    ebp
retn

encode_sub endp
```

==========================================================================================

后记: 文章我是最懒得写的, 写起来真是辛苦, 花了数个小时, 总感觉表述的不是很满意, 无奈已经尽力了

分析水平有限, 错误之处在所难免, 唯恐误人子弟, 读者朋友们见谅了, 希望大家能提出自己的主见.

==========================================================================================

heXer/iPB

2002.10.26
