

# 幻影 v1.5b

> 原文：[`www.pediy.com/kssd/pediy04/kanxue317.htm`](https://www.pediy.com/kssd/pediy04/kanxue317.htm)

幻影 v1.5b 破解应注意的地方   
  
016F:005A6000 9C              PUSHF    
016F:005A6001 55              PUSH    EBP   
016F:005A6002 57              PUSH    EDI   
016F:005A6003 56              PUSH    ESI   
016F:005A6004 52              PUSH    EDX   
016F:005A6005 51              PUSH    ECX   
016F:005A6006 53              PUSH    EBX   
016F:005A6007 9C              PUSHF    
016F:005A6008 FA              CLI      
016F:005A6009 E800000000      CALL    005A600E   
016F:005A600E 5D              POP      EBP   
016F:005A600F 81ED58534000    SUB      EBP,00405358   
016F:005A6015 B085            MOV      AL,85   
016F:005A6017 E800000000      CALL    005A601C   
016F:005A601C 5E              POP      ESI   
016F:005A601D 83C611          ADD      ESI,BYTE +11   
016F:005A6020 B927000000      MOV      ECX,27   
016F:005A6025 3006            XOR      \[ESI\],AL   
016F:005A6027 46              INC      ESI   
016F:005A6028 49              DEC      ECX   
016F:005A6029 75FA            JNZ      005A6025   
016F:005A602B FEC8            DEC      AL   
016F:005A602D B927000000      MOV      ECX,27   
016F:005A6032 3006            XOR      \[ESI\],AL   
016F:005A6034 46              INC      ESI   
016F:005A6035 49              DEC      ECX   
016F:005A6036 75FA            JNZ      005A6032   
016F:005A6038 FEC8            DEC      AL   
016F:005A603A FA              CLI            ；此处设断      
016F:005A603B 56              PUSH    ESI   
016F:005A603C BE65E44000      MOV      ESI,0040E465   
016F:005A6041 03F5            ADD      ESI,EBP   
016F:005A6043 0F010E          SIDT    \[ESI\]   
016F:005A6046 8B7602          MOV      ESI,\[ESI+02\]   
016F:005A6049 66FF4E08        DEC      WORD \[ESI+08\]  ;注意此处陷阱，得想法避开哟   
016F:005A604D FF8DE3D94000    DEC      DWORD \[EBP+0040D9E3\]   
016F:005A6053 5E              POP      ESI   
016F:005A6054 B927000000      MOV      ECX,27   
016F:005A6059 3006            XOR      \[ESI\],AL   
016F:005A605B 46              INC      ESI   
016F:005A605C 49              DEC      ECX   
016F:005A605D 75FA            JNZ      005A6059   
016F:005A605F FEC8            DEC      AL   
016F:005A6061 FA              CLI      
016F:005A6062 56              PUSH    ESI   
016F:005A6063 BE65E44000      MOV      ESI,0040E465   
016F:005A6068 03F5            ADD      ESI,EBP   
016F:005A606A 0F010E          SIDT    \[ESI\]   
016F:005A606D 8B7602          MOV      ESI,\[ESI+02\]   
016F:005A6070 66FF4E08        DEC      WORD \[ESI+08\]   
016F:005A6074 FF8DE3D94000    DEC      DWORD \[EBP+0040D9E3\]   
016F:005A607A 5E              POP      ESI   
  
后面要注意的地方请看 ljtt 的<幻影 v1.5b 分析笔记>  4dd466d0e1e0cc123afbe504da2462ca