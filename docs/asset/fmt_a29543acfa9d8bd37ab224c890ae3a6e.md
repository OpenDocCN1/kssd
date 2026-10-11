
```assembly
; ————————————————————————————

GoOn4:
    esto
    Get OEP:
    log eip
    log T3
    cmp eip, T3
    ja GoOn4
    bpmc
    log eip
    cmt eip, "This is the OEP! Found By: fly"
    MSG "Just : OEP !  Dump and Fix IAT.  Good Luck  "
    ret

NoFind:
    MSG "Error! Maybe It's not tELock V0.80-V0.9X !  "
    ret

4dd466d0e1e0cc123afbe504da2462ca
```
