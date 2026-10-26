
```
004012A4    83E0 02         AND     EAX, 2

004012A7    74 0B           JE      SHORT 004012B4

004012A9    68 68604000     PUSH    00406068                              ; ASCII "Best/Slowest Compression",CR,LF

004012AE    E8 4DFDFFFF     CALL    00401000

004012B3    59              POP     ECX

004012B4    5F              POP     EDI

004012B5    5E              POP     ESI

004012B6    5B              POP     EBX

004012B7    5D              POP     EBP

004012B8    C3              RETN

004015FE    8D45 FF         LEA     EAX, [EBP-1]//restore CC

00401601    50              PUSH    EAX

00401602    FF75 B8         PUSH    DWORD PTR [EBP-48]

00401605    E8 E3FAFFFF     CALL    <save_set_CC>

0040160A    83C4 0C         ADD     ESP, 0C

0040160D    E9 6F020000     JMP     00401881

00401881    C745 88 0200010>MOV     DWORD PTR [EBP-78], 10002             ; UNICODE "::=::\"

00401888    EB 34           JMP     SHORT 004018BE
```
后面就没什么了。

by forgot/iPB

06.05.02

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
