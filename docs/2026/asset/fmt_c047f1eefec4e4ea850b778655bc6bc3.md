
# `Control1Down` proc

```
pushad

invoke    IsDownEnd

.if     eax == 1    ;下面有障碍，改变状态为创建新的方块并保存当前方块在容器中的位置

    mov    curState, 1

    invoke    ChangeAreaState

    invoke    RtlZeroMemory, offset curControl, sizeof curControl

    popad

    ret

.endif

;如果下面不是障碍，则■■■■向下走动

lea    edx, curControl

xor    ecx, ecx
@@:

cmp    ecx, 4

jz    @F

;覆盖原有的方块
push    ecx
invoke    SetColor, 0h, 0h
mov    eax, [edx + ecx * 8 + 4]
mov    ebx, 30
add    ebx, eax    ;ebx右下角y坐标
mov    eax, [edx + ecx * 8]
add    eax, 30        ;eax为右下角x坐标
push    eax        ;保存右下角坐标
push    ebx
push    edx
invoke    Rectangle, hGame, [edx + ecx * 8], [edx + ecx * 8 + 4], eax, ebx
pop    edx
;生成新的方块
invoke    SetColor, 0FF80FFh, 400040h
pop    ebx
pop    eax
pop    ecx
add     DWORD ptr [edx + ecx * 8 + 4], 30
add    ebx, 30
push    ecx
push    edx
invoke    Rectangle, hGame, [edx + ecx * 8], [edx + ecx * 8 + 4], eax, ebx
pop    edx
pop    ecx
inc    ecx
jmp    @B
@@:

popad
ret
```
`Control1Down` endp

# `ChangeAreaState` proc

; 保存当前方块在容器中的状态

```
pushad

;计算区域单元坐标
lea    ebx, curControl
xor    ecx, ecx
@@:

cmp    ecx, 4

jz    @F

;计算二维x坐标
invoke    CalculatePos, [ebx + ecx * 8 ], [ebx + ecx * 8 + 4], 1
mov    esi, eax
;计算二维y坐标
invoke    CalculatePos, [ebx + ecx * 8 ], [ebx + ecx * 8 + 4], 2
mov    edi, eax
;设置区域更新，更新的坐标位置（eax + esi * 4 * 8 + edi * 4）
lea    eax, area
push    eax
xchg    edi, eax
mov    edi, 8
mul    edi
add    esi, eax
pop    eax
mov    DWORD ptr [eax + esi * 4], 1    ;设置当前位置已经被填充
inc    ecx
jmp    @B
@@:

popad
ret
```
`ChangeAreaState` endp

# `CalculatePos` proc

; 根据顶角计算二维坐标
; `left`, `top`：顶角像素
; `posType`：1表示计算x坐标，2表示计算y坐标
; `eax`返回二维坐标

```
left : DWORD, top :DWORD, posType : DWORD
local @ret : DWORD
pushad
mov    edx, left
add    edx, rectArea ;右下角x坐标
mov    ecx, top
add    ecx, rectArea ;右下角y坐标
.if    posType == 1
    mov    eax, edx
    mov    ebx, 30
    xor    edx, edx
    div    ebx
.elseif
    mov    eax, ecx
    sub    eax, topSpace
    mov    ecx, 30
    xor    edx, edx
    div    ecx
.endif
mov    @ret, eax
popad
mov    eax, @ret
dec    eax
ret
```
`CalculatePos` endp

# `IsDownEnd` proc

; 判断下面是否有障碍
; 返回值:：`eax`为1表示下落完毕，为0表示未完毕

```
pushad
lea    ebx, curControl
xor    ecx, ecx
@@:

cmp    ecx, 4

jz    @F

;计算二维x坐标
invoke    CalculatePos, [ebx + ecx * 8 ], [ebx + ecx * 8 + 4], 1
mov    esi, eax
;计算二维y坐标
invoke    CalculatePos, [ebx + ecx * 8 ], [ebx + ecx * 8 + 4], 2
mov    edi, eax
inc    edi
;设置区域更新，更新的坐标位置（eax + esi * 4 * 8 + edi * 4）
lea    eax, area
push    eax
xchg    edi, eax
mov    edi, 8
mul    edi
add    esi, eax
pop    eax
mov    edx, [eax + esi * 4]
.if    edx == 1
    popad
    mov    eax, 1
    ret
.endif
inc    ecx
jmp    @B
@@:

popad
xor    eax, eax
ret
```
`IsDownEnd` endp

## 程序入口

```
start:
    call    _WinMain
    invoke     ExitProcess, NULL
end start
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
