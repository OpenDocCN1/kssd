
```asm
                                        ret  num
                                        reax = -1
                                    ELSE
                                            ret  num
                                    ENDIF
                        ELSEIF (OPATTR (arg)) AND 00000100b        ;; Is an immediate value
                                IF        (OPATTR (arg)) AND 00000001b
                                        mov        eax, arg
                                        ret        num
                                ELSE
                                            IF        arg EQ 0
                                                    xor  eax, eax
                                                    ret  num
                                            ELSEIF         arg EQ 1
                                                xor eax, eax
                                                inc eax
                                                ret num
                                        ELSEIF         arg EQ -1
                                                or eax, NOT 0
                                                ret num
                                        ELSE
                                                      mov eax, arg
                                                      ret num
                                            ENDIF
                                    ENDIF
                                    reax = -1
                          ELSE
                                    mov eax, arg
                                    ret        num
                                    reax = -1
                        ENDIF
                ENDIF
        ENDIF
ENDM
ENDIF
```

### (MF)翻转参数列表

- 用法：`$ArgRev(参数表)`
- 参数：原参数列表(`VARARG`)
- 返回值：翻转后的参数列表
- 影响的寄存器：无

```asm
IFNDEF        $ArgRev
$ArgRev MACRO args:VARARG
LOCAL         arg,y
        y         TEXTEQU         <>
        FOR         arg, <&args>
                y         CATSTR         <arg>, <!,>, y
        ENDM
        y         SUBSTR         y, 1, @SizeStr(%y) - 1
        EXITM         @CatStr(<!<>, y, <!>>)
ENDM
ENDIF
```

### (MA)快速调用

- 用法：`fastcall 函数名, 参数1, 参数2, ....`
- 参数：函数名+参数列表
- 影响的寄存器：`eax`, `ecx`, `edx`(必须改变)

```asm
IFNDEF        fastcall
fastcall MACRO api:REQ, p1, p2, px:VARARG
LOCAL arg, line, recx, reax, redx
        recx = 0
        reax = 0
        redx = 0
        IFNB <px>
                % FOR arg, $ArgRev( <px> )
                        IF        @SizeStr(<arg>) GE 6
                                IFIDNI        @SubStr(<arg>, 1, 5), <ADDR >
                                        lea eax, @SubStr(<arg>, 6)
                                        push eax
                                        reax = -1
                                ELSE
                                        push        arg
                                ENDIF
                        ELSE
                                IFIDNI        <arg>, <eax>        ;;Do not overwrite eax
                                        IF        reax
                                                line TEXTEQU %@Line
                                                % ECHO @FileCur(line) : ERROR! EAX register value overwritten by fastcall macro.
                                                .ERR
                                        ELSE
                                                push arg
                                        ENDIF
                                ELSE
                                        push        arg
                                ENDIF
                        ENDIF
                ENDM
        ENDIF
        IFNB <p1>
                IF        @SizeStr(<p1>) GE 6
                        IFIDNI        @SubStr(<p1>, 1, 5), <ADDR >
%                                lea ecx, @SubStr(<p1>, 6)
                                recx = -1
                        ENDIF
                ENDIF
                IF        @SizeStr(<p1>) GE 8
                        IFIDNI        @SubStr(<p1>, 1, 7), <OFFSET >
                                mov        ecx, p1
                                recx = -1
                        ENDIF
                ENDIF
                IF        @SizeStr(<p1>) GE 10
                        IFIDNI        @SubStr(<p1>, 1, 9), <LROFFSET >
                                mov        ecx, p1
                                recx = -1
                        ENDIF
                ENDIF
                IF         (NOT recx)
                        IF        (OPATTR (p1)) AND 00000100b        ;; Is an immediate value
                                IF        (OPATTR (p1)) AND 00000001b
                                        mov        ecx, p1
                                ELSE
                                        IF        p1 EQ 0
                                                xor ecx, ecx
                                        ELSEIF         p1 EQ 1
                                                xor ecx, ecx
                                                inc ecx
                                        ELSEIF         p1 EQ -1
                                                or ecx, -1
                                        ELSE
                                                mov ecx, p1
                                        ENDIF
                                ENDIF
                                recx = -1
                        ELSEIF        (OPATTR (p1)) AND 00010000b        ;; Is a register value
                                IFDIFI <p1>, <ecx>
                                        IFIDNI <p1>, <eax>
                                                IF        reax
                                                        line TEXTEQU %@Line
                                                        % ECHO @FileCur(line) : ERROR! EAX register value has changed.
                                                        .ERR
                                                ENDIF
                                        ENDIF
                                        mov ecx, p1
                                        recx = -1                                ;; no more ecx
                                ENDIF
                        ELSE
                                mov        ecx, p1
                                recx = -1
                        ENDIF
                ENDIF
        ENDIF
        IFNB <p2>
                IF        @SizeStr(<p2>) GE 6
                        IFIDNI        @SubStr(<p2>, 1, 5), <ADDR >
%                                lea edx, @SubStr(<p2>, 6)
                                redx = -1
                        ENDIF
                ENDIF
                IF        @SizeStr(<p2>) GE 8
                        IFIDNI        @SubStr(<p2>, 1, 7), <OFFSET >
                                mov        edx, p2
                                redx = -1
                        ENDIF
                ENDIF
                IF        @SizeStr(<p2>) GE 10
                        IFIDNI        @SubStr(<p2>, 1, 9), <LROFFSET >
                                mov        edx, p2
                                redx = -1
                        ENDIF
                ENDIF
                IF (NOT redx)
                        IF (OPATTR (p2)) AND 00010000b        ;; Is a register value
                                IFDIFI <p2>, <edx>                ;; do not move edx onto itself
```
