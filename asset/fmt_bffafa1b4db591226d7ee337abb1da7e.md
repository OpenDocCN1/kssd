
```
0356 23 02  00B2F176  load     context.register,01C514DE

0357 12 00  00B2F192  mov      [addr_context.register],addr_context.ebx

0358 19 01  00B2F1D8  push     [context.register]

0359 12 00  00B2F1CA  mov      [addr_context.register],context.esp

0360 11 02  00B2F1BC  sub      [addr_context.register],00000004

0361 01 01  00B2F1A0  pop      [context.register] (4 bytes)

0362 19 01  00B2F1AE  push     [addr_context.register]

0363 14 00  00B2F1F4  pop      [addr_context.esp] (4 bytes)

0364 09 01  00B2F1E6  load     context.register,039132D8

0365 3E 02  00B2F202  mov_reg  [addr_context.eax],context.eax

0366 3E 02  00B2F210  mov_reg  [addr_context.eax],0000047B

0367 12 01  00B2F21E  load     context.register,02793B0E

0368 33 02  00B2F22C  mov      [addr_context.register],addr_context.eax

0369 2D 02  00B2F23A  push     [context.register]

0370 12 00  00B2F248  mov      [addr_context.register],context.ebp

0371 37 02  00B2F264  add      [addr_context.register],034F26A1

0372 32 02  00B2F256  pop      [context.register] (4 bytes)

0373 1B 02  00B2F272  mov      [addr_context.register],context.ebp

0374 0E 01  00B2F2AA  add      [addr_context.register],035D4D61

0375 16 00  00B2F29C  push     [addr_context.register]

0376 1F 01  00B2F28E  mov      [addr_context.register],addr_context.ebx

0377 14 00  00B2F280  pop      [context.register] (4 bytes)

0378 12 00  00B2F2B8  mov      [addr_context.register],context.esp

0379 36 00  00B2F31A  sub      [addr_context.register],00000004

0380 2B 02  00B2F2C6  push     [addr_context.register]

0381 01 01  00B2F2D4  pop      [addr_context.esp] (4 bytes)

0382 2A 00  00B2F30C  push     00B2F336

0383 33 02  00B2F2FE  mov      [addr_context.register],context.esp

0384 32 02  00B2F2E2  pop      [context.register] (4 bytes)

0385 1F 01  00B2F328  mov      [addr_context.register],addr_context.ebx

0386 2A 02  00B2F2F0  call     [context.register]

0387 2C 01  00B2F336  mov      [addr_context.register],context.esp

0388 16 00  00B2F37C  push     [context.register]

0389 1B 02  00B2F36E  mov      [addr_context.register],addr_context.ebx

0390 14 00  00B2F344  pop      [context.register] (4 bytes)

0391 12 00  00B2F352  mov      [addr_context.register],context.esp

0392 26 00  00B2F398  add      [addr_context.register],00000004

0393 2A 00  00B2F360  push     [addr_context.register]

0394 01 01  00B2F38A  pop      [addr_context.esp] (4 bytes)

0395 12 00  00B2F3A6  mov      [addr_context.register],context.esp

0396 16 00  00B2F3EC  push     [context.register]

0397 1B 02  00B2F408  mov      [addr_context.register],addr_context.eax

0398 32 02  00B2F3B4  pop      [context.register] (4 bytes)

0399 33 02  00B2F3D0  mov      [addr_context.register],context.esp

0400 0E 00  00B2F3C2  add      [addr_context.register],00000004

0401 2B 02  00B2F3DE  push     [addr_context.register]

0402 01 01  00B2F3FA  pop      [addr_context.esp] (4 bytes)

0403 2C 01  00B2F416  mov      [addr_context.register],context.ebp

0404 0E 01  00B2F432  add      [addr_context.register],034F0C4D

0405 2D 02  00B2F424  push     [context.register]

0406 2A 00  00B2F440  push     00000000

0407 27 00  00B2F45C  cmp      [esp+4],[esp] (4 bytes)

0408 12 01  00B2F44E  load     context.register,0A4490C7

0409 30 01  00B2F46A  jz       00B2F75E

0410 2C 01  00B2F478  mov      [addr_context.register],addr_context.eax

0411 2B 02  00B2F4CC  push     [context.register]

0412 1F 01  00B2F4A2  mov      [addr_context.register],context.esp

0413 36 00  00B2F486  sub      [addr_context.register],00000004

0414 14 00  00B2F494  pop      [context.register] (4 bytes)

0415 2D 02  00B2F4BE  push     [addr_context.register]

0416 01 01  00B2F4B0  pop      [addr_context.esp] (4 bytes)

0417 1F 01  00B2F4DA  mov      [addr_context.register],addr_context.ebx

0418 2D 02  00B2F520  push     [context.register]

0419 33 02  00B2F52E  mov      [addr_context.register],context.esp

0420 36 00  00B2F504  sub      [addr_context.register],00000004

0421 01 01  00B2F53C  pop      [context.register] (4 bytes)

0422 2B 02  00B2F512  push     [addr_context.register]

0423 01 01  00B2F4E8  pop      [addr_context.esp] (4 bytes)

0424 09 01  00B2F4F6  load     context.register,03BD6BA7

0425 25 01  00B2F54A  mov_reg  [addr_context.eax],0000047B

0426 33 02  00B2F558  mov      [addr_context.register],addr_context.eax

0427 19 01  00B2F566  push     [context.register]

0428 1F 01  00B2F582  mov      [addr_context.register],context.ebp

0429 26 00  00B2F59E  add      [addr_context.register],034F26A1

0430 01 01  00B2F574  pop      [context.register] (4 bytes)

0431 12 01  00B2F590  load     context.register,0B0101A1

0432 33 02  00B2F5AC  mov      [addr_context.register],context.ebp

0433 37 02  00B2F5BA  add      [addr_context.register],035D4A02

0434 2B 02  00B2F5E4  push     [addr_context.register]

0435 1B 02  00B2F5D6  mov      [addr_context.register],addr_context.ebx

0436 01 01  00B2F5C8  pop      [context.register] (4 bytes)

0437 1F 01  00B2F5F2  mov      [addr_context.register],context.esp

0438 36 00  00B2F60E  sub      [addr_context.register],00000004

0439 2B 02  00B2F662  push     [addr_context.register]

0440 32 02  00B2F638  pop      [addr_context.esp] (4 bytes)

0441 2A 00  00B2F646  push     00B2F670

0442 12 00  00B2F600  mov      [addr_context.register],context.esp

0443 01 01  00B2F62A  pop      [context.register] (4 bytes)

0444 1F 01  00B2F654  mov      [addr_context.register],addr_context.ebx

0445 16 01  00B2F61C  call     [context.register]

0446 2C 01  00B2F670  mov      [addr_context.register],context.esp

0447 2B 02  00B2F6A8  push     [context.register]

0448 1B 02  00B2F6D2  mov      [addr_context.register],addr_context.ebx

0449 14 00  00B2F69A  pop      [context.register] (4 bytes)

0450 1F 01  00B2F6C4  mov      [addr_context.register],context.esp

0451 0E 00  00B2F67E  add      [addr_context.register],00000004

0452 2A 00  00B2F68C  push     [addr_context.register]

0453 01 01  00B2F6B6  pop      [addr_context.esp] (4 bytes)

0454 12 00  00B2F6E0  mov      [addr_context.register],context.esp

0455 19 01  00B2F70A  push     [context.register]

0456 12 00  00B2F750  mov      [addr_context.register],addr_context.eax

0457 14 00  00B2F734  pop      [context.register] (4 bytes)

0458 1B 02  00B2F718  mov      [addr_context.register],context.esp

0459 0E 01  00B2F726  add      [addr_context.register],00000004

0460 2B 02  00B2F6FC  push     [addr_context.register]

0461 32 02  00B2F742  pop      [addr_context.esp] (4 bytes)

0462 23 02  00B2F6EE  load     context.register,09EDA0AA

0463 05 00  00B2F75E  mov_reg  [addr_context.eax],context.eax

0464 12 00  00B2F76C  mov      [addr_context.register],context.ebp

0465 37 02  00B2F7A4  add      [addr_context.register],03613FCC

0466 2A 00  00B2F788  push     [context.register]

0467 12 00  00B2F796  mov      [addr_context.register],addr_context.eax

0468 14 00  00B2F77A  pop      [context.register] (4 bytes)

0469 00 02  00B2F7B2  xor_reg  [addr_context.eax],7475EDFF

0470 07 00  00B2F7C0  add_reg  [addr_context.eax],context.ebp

0471 1F 01  00B2F7CE  mov      [addr_context.register],addr_context.eax

0472 06 00  00B2F7DC  jmp      [context.register]

0473 28 02  00B2F7EA  exit_vm
```

还原的代码

```
    ; 第 1 段

    mov     eax,eax
    cmp     dword ptr [ebp+034F2C5D],0
    jnz     l_00B2DF08
    cmp     dword ptr [ebp+034F0C4D],0
    jz      l_00B2E242

l_00B2DF08:

    push    eax
    push    ebx
    mov     eax,eax
    mov     eax,0000047B
    mov     [ebp+034F1B41],eax
```
