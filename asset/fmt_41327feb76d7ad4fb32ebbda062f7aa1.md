
```
.text:0040120C                    or      ecx, 0FFFFFFFFh
.text:0040120F                    div     ecx
.text:00401211                    mov     edx, [ebp+a]
.text:00401214                    add     eax, edx
.text:00401216                    xor     edx, edx
.text:00401218                    div     [ebp+f2]
.text:0040121B                    sub     eax, 1
.text:0040121E                    xor     edx, edx
.text:00401220                    or      ecx, 0FFFFFFFFh
.text:00401223                    div     ecx
.text:00401225                    mov     ecx, eax
.text:00401227                    imul    ecx, 9
.text:0040122A                    add     ecx, 1
.text:0040122D                    mov     eax, [ebp+f2]
.text:00401230                    xor     edx, edx
.text:00401232                    div     ecx
.text:00401234                    mov     [ebp+f2], eax
.text:00401237                    mov     eax, [ebp+a]
.text:0040123A                    sub     eax, 1
.text:0040123D                    xor     edx, edx
.text:0040123F                    or      ecx, 0FFFFFFFFh
.text:00401242                    div     ecx
.text:00401244                    mov     edx, [ebp+a]
.text:00401247                    add     eax, edx
.text:00401249                    xor     edx, edx
.text:0040124B                    div     [ebp+f2]
.text:0040124E                    sub     eax, 1
.text:00401251                    xor     edx, edx
.text:00401253                    or      ecx, 0FFFFFFFFh
.text:00401256                    div     ecx
.text:00401258                    imul    eax, 9
.text:0040125B                    add     eax, 1
.text:0040125E                    mov     edx, [ebp+f1]
.text:00401261                    imul    edx, eax
.text:00401264                    mov     [ebp+f1], edx
.text:00401267                    mov     eax, [ebp+a]
.text:0040126A                    sub     eax, 1
.text:0040126D                    xor     edx, edx
.text:0040126F                    or      ecx, 0FFFFFFFFh
.text:00401272                    div     ecx
.text:00401274                    mov     edx, [ebp+a]
.text:00401277                    add     eax, edx
.text:00401279                    xor     edx, edx
.text:0040127B                    div     [ebp+f2]
.text:0040127E                    sub     eax, 1
.text:00401281                    xor     edx, edx
.text:00401283                    or      ecx, 0FFFFFFFFh
.text:00401286                    div     ecx
.text:00401288                    mov     ecx, eax
.text:0040128A                    imul    ecx, 9
.text:0040128D                    add     ecx, 1
.text:00401290                    mov     eax, [ebp+f2]
.text:00401293                    xor     edx, edx
.text:00401295                    div     ecx
.text:00401297                    mov     [ebp+f2], eax
.text:0040129A                    mov     eax, [ebp+a]
.text:0040129D                    sub     eax, 1
.text:004012A0                    xor     edx, edx
.text:004012A2                    or      ecx, 0FFFFFFFFh
.text:004012A5                    div     ecx
.text:004012A7                    mov     edx, [ebp+a]
.text:004012AA                    add     eax, edx
.text:004012AC                    xor     edx, edx
.text:004012AE                    div     [ebp+f2]
.text:004012B1                    sub     eax, 1
.text:004012B4                    xor     edx, edx
.text:004012B6                    or      ecx, 0FFFFFFFFh
.text:004012B9                    div     ecx
.text:004012BB                    imul    eax, 9
.text:004012BE                    add     eax, 1
.text:004012C1                    mov     edx, [ebp+f1]
.text:004012C4                    imul    edx, eax
.text:004012C7                    mov     [ebp+f1], edx
.text:004012CA                    mov     eax, [ebp+a]
.text:004012CD                    sub     eax, 1
.text:004012D0                    xor     edx, edx
.text:004012D2                    or      ecx, 0FFFFFFFFh
.text:004012D5                    div     ecx
.text:004012D7                    mov     edx, [ebp+a]
.text:004012DA                    add     eax, edx
.text:004012DC                    xor     edx, edx
.text:004012DE                    div     [ebp+f2]
.text:004012E1                    sub     eax, 1
.text:004012E4                    xor     edx, edx
.text:004012E6                    or      ecx, 0FFFFFFFFh
.text:004012E9                    div     ecx
.text:004012EB                    mov     ecx, eax
.text:004012ED                    imul    ecx, 9
.text:004012F0                    add     ecx, 1
.text:004012F3                    mov     eax, [ebp+f2]
.text:004012F6                    xor     edx, edx
.text:004012F8                    div     ecx
.text:004012FA                    mov     [ebp+f2], eax
.text:004012FD                    mov     eax, [ebp+a]
.text:00401300                    sub     eax, 1
.text:00401303                    xor     edx, edx
.text:00401305                    or      ecx, 0FFFFFFFFh
.text:00401308                    div     ecx
.text:0040130A                    mov     edx, [ebp+a]
.text:0040130D                    add     eax, edx
.text:0040130F                    xor     edx, edx
.text:00401311                    div     [ebp+f2]
.text:00401314                    sub     eax, 1
.text:00401317                    xor     edx, edx
.text:00401319                    or      ecx, 0FFFFFFFFh
.text:0040131C                    div     ecx
.text:0040131E                    imul    eax, 9
.text:00401321                    add     eax, 1
.text:00401324                    mov     edx, [ebp+f1]
.text:00401327                    imul    edx, eax
.text:0040132A                    mov     [ebp+f1], edx
.text:0040132D                    mov     eax, [ebp+a]
.text:00401330                    sub     eax, 1
.text:00401333                    xor     edx, edx
.text:00401335                    or      ecx, 0FFFFFFFFh
.text:00401338                    div     ecx
.text:0040133A                    mov     edx, [ebp+a]
.text:0040133D                    add     eax, edx
.text:0040133F                    xor     edx, edx
.text:00401341                    div     [ebp+f2]
.text:00401344                    sub     eax, 1
.text:00401347                    xor     edx, edx
.text:00401349                    or      ecx, 0FFFFFFFFh
.text:0040134C                    div     ecx
.text:0040134E                    mov     ecx, eax
.text:00401350                    imul    ecx, 9
.text:00401353                    add     ecx, 1
.text:00401356                    mov     eax, [ebp+f2]
.text:00401359                    xor     edx, edx
.text:0040135B                    div     ecx
.text:0040135D                    mov     [ebp+f2], eax
.text:00401360                    mov     eax, [ebp+a]
.text:00401363                    sub     eax, 1
.text:00401366                    xor     edx, edx
.text:00401368                    or      ecx, 0FFFFFFFFh
.text:0040136B                    div     ecx
.text:0040136D                    mov     edx, [ebp+a]
.text:00401370                    add     eax, edx
.text:00401372                    xor     edx, edx
.text:00401374                    div     [ebp+f2]
.text:00401377                    sub     eax, 1
.text:0040137A                    xor     edx, edx
.text:0040137C                    or      ecx, 0FFFFFFFFh
.text:0040137F                    div     ecx
.text:00401381                    imul    eax, 9
.text:00401384                    add     eax, 1
.text:00401387                    mov     edx, [ebp+f1]
```
