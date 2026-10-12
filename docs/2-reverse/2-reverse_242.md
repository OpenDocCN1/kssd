
# Softice 宏定义

> 原文：[`www.pediy.com/kssd/pediy03/forum190.htm`](https://www.pediy.com/kssd/pediy03/forum190.htm)

```
MACRO sname=" s 0 l FFFFFFFF 'Tennight.C' "   

MACRO sserial=" s 0 l FFFFFFFF '13131313' "   

MACRO wideser=" s 0 l FFFFFFFF '1',0,'3',0,'1',0,'3',0,'1',0,'3',0,'1',0,'3' "   

MACRO widename=" s 0 l FFFFFFFF 'T',0,'e',0,'n',0,'n',0,'i',0,'g',0,'h',0,'t',0,'.',0,'C' "   

MACRO argdlg="D esp->c ; p ret"   

MACRO argwin="D esp->8 ; p ret"   

MACRO argreg="D ESP->14;"   

MACRO argpe="D ESP->4;"   

MACRO bpxget=" bpx GetDlgItemTextA DO \\" argdlg \\" ; bpx GetWindowTextA DO \\" argwin \\" "   

MACRO bpxget16=" bpx GetDlgItemText; bpx GetWindowText "   

MACRO bpxgeta=" bpx GetDlgItemTextA; bpx getwindowtexta; bpx getdlgitemint; bpx getdlgitemtext; bpx getwindowtext; bpx getwindowtextw; bpx getdlgitemtextw "   

MACRO reg="bpx RegQueryValueExA IF \*(ESP->8) == '%1' DO \\" argreg \\" "   

MACRO bpxpe=" bpx LoadLibraryA DO \\" argpe \\" "   

MACRO anti=" bpx CreatFileA DO \\" argpe \\" "   4dd466d0e1e0cc123afbe504da2462ca
```
