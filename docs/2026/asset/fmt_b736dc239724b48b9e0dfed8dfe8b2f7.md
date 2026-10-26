
```
    _KeyGen       proc          uses ebx esi edi _lpszUserName, _lpszSerial

                  ; 生成序列号
                  ; 入口参数: _lpszUserName——指向给定的用户名的指针
                  ; 出口参数: _lpszSerial——指向所生成序列号的缓冲区指针
                  ; 返回值: eax——等于1表示成功，0表示失败

                  mov           esi, _lpszUserName
                  invoke        lstrlen, esi
                  mov           ebx, eax
                  mov           ecx, 10
                  mov           edi, _lpszSerial
                  .if           eax < 17

                                xor        eax, eax
                                jmp        locret_1

                  .elseif       eax < 20
                                movsx      eax, byte ptr [esi+ebx-3]
                                inc        eax
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+9]
                                add        eax, 7
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+2]
                                movsx      edx, byte ptr [esi+ebx-2]
                                add        eax, edx
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+7]
                                add        eax, 3
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+4]
                                cdq
                                idiv       ecx
                                inc        edx
                                movsx      eax, byte ptr [esi+edx]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+10]
                                add        eax, 5
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+ebx-6]
                                add        eax, ebx
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+16]
                                add        eax, 36
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                  .elseif       eax < 26
                                movsx      eax, byte ptr [esi+17]
                                movsx      edx, byte ptr [esi+ebx-7]
                                add        eax, edx
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+ebx-10]
                                add        eax, 3
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+7]
                                cdq
                                idiv       ecx
                                movsx      eax, byte ptr [esi+edx+3]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+3]
                                add        eax, 19
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+11]
                                add        eax, 2
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+4]
                                add        eax, ebx
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi]
                                movsx      edx, byte ptr [esi+ebx-1]
                                add        eax, edx
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+13]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                  .elseif       eax < 34
                                movsx      eax, byte ptr [esi+ebx-1]
                                add        eax, 2
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+8]
                                add        eax, 2
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+ebx-9]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+1]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+13]
                                lea        eax, [eax+ebx-3]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+23]
                                add        eax, 7
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+5]
                                add        eax, ebx
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+16]
                                sub        eax, 2
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                  .else
                                movsx      eax, byte ptr [esi+ebx-5]
                                sub        eax, 2
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+7]
                                add        eax, 2
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi]
                                movsx      edx, byte ptr [esi+ebx-1]
                                lea        eax, [eax+edx-2]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+14]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+20]
                                add        eax, 6
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+8]
                                inc        eax
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+ebx-10]
                                sub        eax, 3
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                                movsx      eax, byte ptr [esi+17]
                                lea        eax, [eax+ebx+1]
                                cdq
                                idiv       ecx
                                lea        eax, [edx+30h]
                                stosb

                  .endif
                  xor           eax, eax
                  stosb
                  inc           eax

          locret_1:
                  ret

    _KeyGen       endp
```
