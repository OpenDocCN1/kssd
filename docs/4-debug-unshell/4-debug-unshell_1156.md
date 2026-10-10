

# VGCrypt PE Encryptor V0.75

> 原文：[`www.pediy.com/kssd/pediy06/pediy6258.htm`](https://www.pediy.com/kssd/pediy06/pediy6258.htm)

VGCrypt PE Encryptor V0.75 脱壳——Vgcrypt.exe 主程序   
    
    
    
下载地址：  http://member.netease.com/~fsdb/source/vgcrypt.zip    
软件大小：  16 KB   
  
  
【软件简介】： This is a fairly simple PE encryptor I wrote up. I commented everything that is relavent to PE appendation or insertion, more so than I needed to even. The most interesting feature of this encryptor is that it attempts to find a location to insert itself between object virtual size and the next file alignment boundary, thus not changing the physical file size.   
  
【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！   
  
【破解工具】：Ollydbg1.09、PEiD、LordPE、ImportREC   
  
—————————————————————————————————    
【脱壳过程】：   
             
            
调试环境：Win98SE + Ollydbg     这个 V0.75 的 VGCrypt 有脱壳机：UnVGCrypt V0.1   
  
呵呵，Vgcrypt.exe 主程序真行，居然自己给自己加了 4 层壳！:-)   
————————————————————————   
一、第 1 层壳   
  
  
00408000     9C                   pushfd   
                                  ====>进入 OD 后断在这！   
  
00408001     55                   push ebp   
00408002     E8 EC000000          call VGCRYPT.004080F3   
                                  ====>F7 进入   
  
004080F3     E8 00000000          call VGCRYPT.004080F8   
                                  ====>变形 JMP！F7 走进   
  
004080F8     8B2C24               mov ebp,dword ptr ss:\[esp\]   
004080FB     81ED 1C274000        sub ebp,VGCRYPT.0040271C   
00408101     83C4 04              add esp,4   
00408104     C3                   retn   
                                  ====>返回到 00408007   
  
00408007     87D5                 xchg ebp,edx   
00408009     5D                   pop ebp   
0040800A     60                   pushad   
0040800B     87D5                 xchg ebp,edx   
0040800D     80BD 15274000 01     cmp byte ptr ss:\[ebp+402715\],1   
00408014     74 39                je short VGCRYPT.0040804F   
00408016     C685 15274000 01     mov byte ptr ss:\[ebp+402715\],1   
0040801D     E9 E4000000          jmp VGCRYPT.00408106   
                                  ====>跳   
  
00408106     E8 1B000000          call VGCRYPT.00408126   
                                  ====>F7 进入   
  
00408126     64:67:FF36 0000      push dword ptr fs:\[0\]   
0040812C     64:67:8926 0000      mov dword ptr fs:\[0\],esp   
00408132     8DB5 47264000        lea esi,dword ptr ss:\[ebp+402647\]   
00408138     8BFE                 mov edi,esi   
0040813A     B9 CA000000          mov ecx,0CA   
0040813F     8AA5 11274000        mov ah,byte ptr ss:\[ebp+402711\]   
00408145     AC                   lods byte ptr ds:\[esi\]   
00408146     32C4                 xor al,ah   
00408148     FEC4                 inc ah   
0040814A     C0C4 02              rol ah,2   
0040814D     80C4 90              add ah,90   
00408150     AA                   stos byte ptr es:\[edi\]   
00408151   ^ E2 F2                loopd short VGCRYPT.00408145   
                                  ====>F4 下去，跳出 LOOP！   
  
00408153   ^ E9 CBFEFFFF          jmp VGCRYPT.00408023   
                                  ====>跳   
  
00408023     E8 00000000          call VGCRYPT.00408028   
                                  ====>变形 JMP！F7 走进   
  
00408028     8B9D 05274000        mov ebx,dword ptr ss:\[ebp+402705\]   
0040802E     83C3 28              add ebx,28   
00408031     58                   pop eax   
00408032     2BC3                 sub eax,ebx   
00408034     8985 0D274000        mov dword ptr ss:\[ebp+40270D\],eax   
0040803A     CC                   int3   
                                  ====>F4 下去！避开这个 int3，否则程序运行！   
  
0040803B     8DBD 24264000        lea edi,dword ptr ss:\[ebp+402624\]   
00408041     B9 3B000000          mov ecx,3B   
00408046     F3:AA                rep stos byte ptr es:\[edi\]   
00408048     64:67:8F06 0000      pop dword ptr fs:\[0\]   
0040804E     5A                   pop edx   
0040804F     8B85 0D274000        mov eax,dword ptr ss:\[ebp+40270D\]   
00408055     0185 09274000        add dword ptr ss:\[ebp+402709\],eax   
0040805B     61                   popad   
0040805C     9D                   popfd   
0040805D     8B9A 09274000        mov ebx,dword ptr ds:\[edx+402709\]   
00408063     898A 09274000        mov dword ptr ds:\[edx+402709\],ecx   
                                  ====>F4 直接到这！   
  
00408069     FFE3                 jmp ebx   
                                  ====>跳至 00407000   
                                  ====>一般 Vgcrypt 加壳这里就跳向 OEP 了！   
  
  
————————————————————————   
二、第 2 层壳   
  
  
00407000     9C                   pushfd   
00407001     55                   push ebp   
00407002     E8 EC000000          call VGCRYPT.004070F3   
                                  ====>F7 进入   
  
004070F3     E8 00000000          call VGCRYPT.004070F8   
                                  ====>变形 JMP！F7 走进   
  
004070F8     8B2C24               mov ebp,dword ptr ss:\[esp\]   
004070FB     81ED 1C274000        sub ebp,VGCRYPT.0040271C   
00407101     83C4 04              add esp,4   
00407104     C3                   retn   
                                  ====>返回到 00407007   
  
00407007     87D5                 xchg ebp,edx   
00407009     5D                   pop ebp   
0040700A     60                   pushad   
0040700B     87D5                 xchg ebp,edx   
0040700D     80BD 15274000 01     cmp byte ptr ss:\[ebp+402715\],1   
00407014     74 39                je short VGCRYPT.0040704F   
00407016     C685 15274000 01     mov byte ptr ss:\[ebp+402715\],1   
0040701D     E9 E4000000          jmp VGCRYPT.00407106   
                                  ====>跳   
  
00407106     E8 1B000000          call VGCRYPT.00407126   
                                  ====>F7 进入   
  
00407126     64:67:FF36 0000      push dword ptr fs:\[0\]   
0040712C     64:67:8926 0000      mov dword ptr fs:\[0\],esp   
00407132     8DB5 47264000        lea esi,dword ptr ss:\[ebp+402647\]   
00407138     8BFE                 mov edi,esi   
0040713A     B9 CA000000          mov ecx,0CA   
0040713F     8AA5 11274000        mov ah,byte ptr ss:\[ebp+402711\]   
00407145     AC                   lods byte ptr ds:\[esi\]   
00407146     32C4                 xor al,ah   
00407148     FEC4                 inc ah   
0040714A     C0C4 02              rol ah,2   
0040714D     80C4 90              add ah,90   
00407150     AA                   stos byte ptr es:\[edi\]   
00407151   ^ E2 F2                loopd short VGCRYPT.00407145   
                                  ====>F4 下去，跳出 LOOP！   
  
00407153   ^ E9 CBFEFFFF          jmp VGCRYPT.00407023   
                                  ====>跳   
  
00407023     E8 00000000          call VGCRYPT.00407028   
00407028     8B9D 05274000        mov ebx,dword ptr ss:\[ebp+402705\]   
0040702E     83C3 28              add ebx,28   
00407031     58                   pop eax   
00407032     2BC3                 sub eax,ebx   
00407034     8985 0D274000        mov dword ptr ss:\[ebp+40270D\],eax   
0040703A     CC                   int3   
                                  ====>F4 下去！避开这个 int3，否则程序运行！   
  
0040703B     8DBD 24264000        lea edi,dword ptr ss:\[ebp+402624\]   
00407041     B9 3B000000          mov ecx,3B   
00407046     F3:AA                rep stos byte ptr es:\[edi\]   
00407048     64:67:8F06 0000      pop dword ptr fs:\[0\]   
0040704E     5A                   pop edx   
0040704F     8B85 0D274000        mov eax,dword ptr ss:\[ebp+40270D\]   
00407055     0185 09274000        add dword ptr ss:\[ebp+402709\],eax   
0040705B     61                   popad   
0040705C     9D                   popfd   
0040705D     8B9A 09274000        mov ebx,dword ptr ds:\[edx+402709\]   
00407063     898A 09274000        mov dword ptr ds:\[edx+402709\],ecx   
                                  ====>F4 直接到这！   
  
00407069     FFE3                 jmp ebx   
                                  ====>跳至 00406000    
  
  
————————————————————————   
三、第 3 层壳   
  
  
下面采用更快的方法！ :-)  都是一样的流程，按上面的方法走也可以。   
  
00406000     9C                   pushfd   
00406001     55                   push ebp   
00406002     E8 EC000000          call VGCRYPT.004060F3   
                                  ====>这次不进入啦！我有更好的方法  ^O^   
  
00406007     87D5                 xchg ebp,edx   
00406009     5D                   pop ebp   
0040600A     60                   pushad   
0040600B     87D5                 xchg ebp,edx   
0040600D     80BD 15274000 01     cmp byte ptr ss:\[ebp+402715\],1   
00406014     74 39                je short VGCRYPT.0040604F   
00406016     C685 15274000 01     mov byte ptr ss:\[ebp+402715\],1   
0040601D     E9 E4000000          jmp VGCRYPT.00406106   
                                  ====>跳  呵呵，省了一段路程   
    
00406106     E8 1B000000          call VGCRYPT.00406126   
0040610B     8B6424 08            mov esp,dword ptr ss:\[esp+8\]   
0040610F     E8 DFFFFFFF          call VGCRYPT.004060F3   
00406114     C685 CC264000 C3     mov byte ptr ss:\[ebp+4026CC\],0C3   
0040611B     E8 4BFFFFFF          call VGCRYPT.0040606B   
00406120   ^ E9 16FFFFFF          jmp VGCRYPT.0040603B   
00406125     B7 64                mov bh,64   
00406127     67:FF36 0000         push dword ptr ds:\[0\]   
0040612C     64:67:8926 0000      mov dword ptr fs:\[0\],esp   
00406132     8DB5 47264000        lea esi,dword ptr ss:\[ebp+402647\]   
00406138     8BFE                 mov edi,esi   
0040613A     B9 CA000000          mov ecx,0CA   
0040613F     8AA5 11274000        mov ah,byte ptr ss:\[ebp+402711\]   
00406145     AC                   lods byte ptr ds:\[esi\]   
00406146     32C4                 xor al,ah   
00406148     FEC4                 inc ah   
0040614A     C0C4 02              rol ah,2   
0040614D     80C4 90              add ah,90   
00406150     AA                   stos byte ptr es:\[edi\]   
00406151   ^ E2 F2                loopd short VGCRYPT.00406145   
00406153   ^ E9 CBFEFFFF          jmp VGCRYPT.00406023   
                                  ====>直接 F4 到这，跳出 LOOP！   
  
00406023     E8 00000000          call VGCRYPT.00406028   
00406028     8B9D 05274000        mov ebx,dword ptr ss:\[ebp+402705\]   
0040602E     83C3 28              add ebx,28   
00406031     58                   pop eax   
00406032     2BC3                 sub eax,ebx   
00406034     8985 0D274000        mov dword ptr ss:\[ebp+40270D\],eax   
0040603A     CC                   int3   
0040603B     8DBD 24264000        lea edi,dword ptr ss:\[ebp+402624\]   
00406041     B9 3B000000          mov ecx,3B   
00406046     F3:AA                rep stos byte ptr es:\[edi\]   
00406048     64:67:8F06 0000      pop dword ptr fs:\[0\]   
0040604E     5A                   pop edx   
0040604F     8385 0D274000 01     add dword ptr ss:\[ebp+40270D\],1   
00406056     8509                 test dword ptr ds:\[ecx\],ecx   
00406058     27                   daa   
00406059     40                   inc eax   
0040605A     0061 9D              add byte ptr ds:\[ecx\-63\],ah   
0040605D     8B9A 09274000        mov ebx,dword ptr ds:\[edx+402709\]   
00406063     898A 09274000        mov dword ptr ds:\[edx+402709\],ecx   
                                  ====>F4 直接到这！   
  
00406069     FFE3                 jmp ebx   
                                  ====>跳至 00405000   
  
  
————————————————————————   
四、第 4 层壳   
  
  
00405000     9C                   pushfd   
00405001     55                   push ebp   
00405002     E8 EC000000          call VGCRYPT.004050F3   
00405007     87D5                 xchg ebp,edx   
00405009     5D                   pop ebp   
0040500A     60                   pushad   
0040500B     87D5                 xchg ebp,edx   
0040500D     80BD 15274000 01     cmp byte ptr ss:\[ebp+402715\],1   
00405014     74 39                je short VGCRYPT.0040504F   
00405016     C685 15274000 01     mov byte ptr ss:\[ebp+402715\],1   
0040501D     E9 E4000000          jmp VGCRYPT.00405106   
                                  ====>跳    
  
00405106     E8 1B000000          call VGCRYPT.00405126   
0040510B     8B6424 08            mov esp,dword ptr ss:\[esp+8\]   
0040510F     E8 DFFFFFFF          call VGCRYPT.004050F3   
00405114     C685 CC264000 C3     mov byte ptr ss:\[ebp+4026CC\],0C3   
0040511B     E8 4BFFFFFF          call VGCRYPT.0040506B   
00405120   ^ E9 16FFFFFF          jmp VGCRYPT.0040503B   
00405125     B7 64                mov bh,64   
00405127     67:FF36 0000         push dword ptr ds:\[0\]   
0040512C     64:67:8926 0000      mov dword ptr fs:\[0\],esp   
00405132     8DB5 47264000        lea esi,dword ptr ss:\[ebp+402647\]   
00405138     8BFE                 mov edi,esi   
0040513A     B9 CA000000          mov ecx,0CA   
0040513F     8AA5 11274000        mov ah,byte ptr ss:\[ebp+402711\]   
00405145     AC                   lods byte ptr ds:\[esi\]   
00405146     32C4                 xor al,ah   
00405148     FEC4                 inc ah   
0040514A     C0C4 02              rol ah,2   
0040514D     80C4 90              add ah,90   
00405150     AA                   stos byte ptr es:\[edi\]   
00405151   ^ E2 F2                loopd short VGCRYPT.00405145   
00405153   ^ E9 CBFEFFFF          jmp VGCRYPT.00405023   
                                  ====>直接 F4 到这，跳出 LOOP！   
  
00405023     E8 00000000          call VGCRYPT.00405028   
00405028     8B9D 05274000        mov ebx,dword ptr ss:\[ebp+402705\]   
0040502E     83C3 28              add ebx,28   
00405031     58                   pop eax   
00405032     2BC3                 sub eax,ebx   
00405034     8985 0D274000        mov dword ptr ss:\[ebp+40270D\],eax   
0040503A     CC                   int3   
0040503B     8DBD 24264000        lea edi,dword ptr ss:\[ebp+402624\]   
00405041     B9 3B000000          mov ecx,3B   
00405046     F3:AA                rep stos byte ptr es:\[edi\]   
00405048     64:67:8F06 0000      pop dword ptr fs:\[0\]   
0040504E     5A                   pop edx   
0040504F     8B85 0D274000        mov eax,dword ptr ss:\[ebp+40270D\]   
00405055     0185 09274000        add dword ptr ss:\[ebp+402709\],eax   
0040505B     61                   popad   
0040505C     9D                   popfd   
0040505D     8B9A 09274000        mov ebx,dword ptr ds:\[edx+402709\]   
00405063     898A 09274000        mov dword ptr ds:\[edx+402709\],ecx   
                                  ====>F4 直接到这！   
  
00405069     FFE3                 jmp ebx   
                                  ====>跳至 00401000  这就是 OEP 值  :-)   
  
  
————————————————————————   
00401000       E8                 db E8   
                                  ====>在这儿用 LordPE 完全 DUMP 这个进程   
  
00401001       51                 db 51     
00401002       06                 db 06   
  
  
———————————————————————   
  
停在 OEP 处，运行 ImportREC，选择这个进程。把 OEP 改为 00001000，点 IT AutoSearch，点“Get Import”，   
函数都是有效的。FixDump，正常运行！  8.5K ->40K   
    
  
—————————————————————————————————   
       
                                   
         ,     \_/    
        /| \_.-~/            \_     ,        青春都一饷   
       ( /~   /              ~-.\_ |   
       \`\\  \_/                   ~ )          忍把浮名    
   \_-~~~-.)  )\_\_/;;,.          \_  //'    
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`    
  /~~//'   /' \`~         ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~      
                         "     "   "~'  ""   
  
       
  
               Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]   
  
                       2003-10-02  02:20 

  
 4dd466d0e1e0cc123afbe504da2462ca