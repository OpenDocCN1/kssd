
```
006132D1    8B85 ACFEFFFF   mov eax,dword ptr ss:[ebp-154]  

006132D7    E8 C877DFFF      call 0040AAA4  

006132DC    8B85 78FEFFFF   mov eax,dword ptr ss:[ebp-188]  

006132E2    8D95 7CFEFFFF   lea edx,dword ptr ss:[ebp-184]  

006132E8    E8 6367DFFF      call 00409A50  

006132ED    8B85 7CFEFFFF   mov eax,dword ptr ss:[ebp-184]  

006132F3    BA A4356100      mov edx,006135A4 ; ASCII "STOCKSTORM.EXE"  

006132F8    E8 2B20DFFF      call 00405328  

//是否是 STOCKSTORM.EXE  

006132FD    74 34            je short 00613333  

006132FF    8D95 70FEFFFF   lea edx,dword ptr ss:[ebp-190]  

00613305    8B85 ACFEFFFF   mov eax,dword ptr ss:[ebp-154]  

0061330B    E8 9477DFFF      call 0040AAA4  

00613310    8B85 70FEFFFF   mov eax,dword ptr ss:[ebp-190]  

00613316    8D95 74FEFFFF   lea edx,dword ptr ss:[ebp-18C]  

0061331C    E8 2F67DFFF      call 00409A50  

00613321    8B85 74FEFFFF   mov eax,dword ptr ss:[ebp-18C]  

00613327    BA BC356100      mov edx,006135BC ; ASCII "UPDATE.EXE"  

0061332C    E8 F71FDFFF      call 00405328  

//是否是 UPDATE.EXE  

00613331    75 0C            jnz short 0061333F  

00613333    81BD A4FEFFFF 0>cmp dword ptr ss:[ebp-15C],19000  

0061333D    7D 66            jge short 006133A5  

0061333F    8D95 68FEFFFF   lea edx,dword ptr ss:[ebp-198]  

00613345    A1 70096200      mov eax,dword ptr ds:[620970]  

0061334A    8B00             mov eax,dword ptr ds:[eax]  

0061334C    E8 83A2E7FF      call 0048D5D4  

00613351    8B85 68FEFFFF   mov eax,dword ptr ss:[ebp-198]  

00613357    8D95 6CFEFFFF   lea edx,dword ptr ss:[ebp-194]  

0061335D    E8 BA76DFFF      call 0040AA1C  

00613362    8D85 6CFEFFFF   lea eax,dword ptr ss:[ebp-194]  

00613368    8B95 ACFEFFFF   mov edx,dword ptr ss:[ebp-154]  

0061336E    E8 711EDFFF      call 004051E4  

00613373    8B85 6CFEFFFF   mov eax,dword ptr ss:[ebp-194]  

00613379    E8 3614EEFF      call 004F47B4  

//如果目录下有非 STOCKSTORM.EXE 和 UPDATE.EXE 之外的其他 exe 文件，对不起，删之
```

下面就是在 C 盘下创建一个批处理文件，格式如下：

```
:try  

del "G:\UnPack\ACPrtect\股市风暴 V6.0\UnPacKed.exe"  

if exist "G:\UnPack\ACPrtect\股市风暴 V6.0\UnPacKed.exe" goto try  

del %0
```

```
004F480F    B9 AC494F00      mov ecx,004F49AC ; ASCII ".bat"  

004F4814    E8 0F0AF1FF      call 00405228  

004F4819    8D45 F8          lea eax,dword ptr ss:[ebp-8]  

004F481C    8B4D F4          mov ecx,dword ptr ss:[ebp-C]  

004F481F    BA BC494F00      mov edx,004F49BC  

004F4824    E8 FF09F1FF      call 00405228  

004F4829    8B55 F8          mov edx,dword ptr ss:[ebp-8]  

004F482C    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F4832    E8 8DE7F0FF      call 00402FC4  

004F4837    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F483D    E8 1EE5F0FF      call 00402D60  

004F4842    E8 71E1F0FF      call 004029B8  

004F4847    BA C8494F00      mov edx,004F49C8 ; ASCII ":try"  

004F484C    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F4852    E8 A10DF1FF      call 004055F8  

004F4857    E8 7CF1F0FF      call 004039D8  

004F485C    E8 57E1F0FF      call 004029B8  

004F4861    68 D8494F00      push 004F49D8 ; ASCII "del ""  

004F4866    FF75 FC          push dword ptr ss:[ebp-4]  

004F4869    68 E8494F00      push 004F49E8  

004F486E    8D85 CCFDFFFF   lea eax,dword ptr ss:[ebp-234]  

004F4874    BA 03000000      mov edx,3  

004F4879    E8 1E0AF1FF      call 0040529C  

004F487E    8B95 CCFDFFFF   mov edx,dword ptr ss:[ebp-234]  

004F4884    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F488A    E8 690DF1FF      call 004055F8  

004F488F    E8 44F1F0FF      call 004039D8  

004F4894    E8 1FE1F0FF      call 004029B8  

004F4899    68 F4494F00      push 004F49F4 ; ASCII "if exist ""  

004F489E    FF75 FC          push dword ptr ss:[ebp-4]  

004F48A1    68 E8494F00      push 004F49E8  

004F48A6    68 084A4F00      push 004F4A08 ; ASCII " goto try"  

004F48AB    8D85 C8FDFFFF   lea eax,dword ptr ss:[ebp-238]  

004F48B1    BA 04000000      mov edx,4  

004F48B6    E8 E109F1FF      call 0040529C  

004F48BB    8B95 C8FDFFFF   mov edx,dword ptr ss:[ebp-238]  

004F48C1    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F48C7    E8 2C0DF1FF      call 004055F8  

004F48CC    E8 07F1F0FF      call 004039D8  

004F48D1    E8 E2E0F0FF      call 004029B8  

004F48D6    BA 1C4A4F00      mov edx,004F4A1C ; ASCII "del %0"  

004F48DB    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F48E1    E8 120DF1FF      call 004055F8  

004F48E6    E8 EDF0F0FF      call 004039D8  

004F48EB    E8 C8E0F0FF      call 004029B8  

004F48F0    8D85 28FEFFFF   lea eax,dword ptr ss:[ebp-1D8]  

004F48F6    E8 45E8F0FF      call 00403140  

004F48FB    E8 B8E0F0FF      call 004029B8  

004F4900    8D85 D4FDFFFF   lea eax,dword ptr ss:[ebp-22C]  

004F4906    33C9             xor ecx,ecx  

004F4908    BA 44000000      mov edx,44  

004F490D    E8 C2EAF0FF      call 004033D4  

004F4912    C785 00FEFFFF 0>mov dword ptr ss:[ebp-200],1  

004F491C    66:C785 04FEFFF>mov word ptr ss:[ebp-1FC],0  

004F4925    8D85 18FEFFFF   lea eax,dword ptr ss:[ebp-1E8]  

004F492B    50               push eax  

004F492C    8D85 D4FDFFFF   lea eax,dword ptr ss:[ebp-22C]  

004F4932    50               push eax  

004F4933    6A 00            push 0  

004F4935    6A 00            push 0  

004F4937    6A 40            push 40  

004F4939    6A 00            push 0  

004F493B    6A 00            push 0  

004F493D    6A 00            push 0  

004F493F    8B45 F8          mov eax,dword ptr ss:[ebp-8]  

004F4942    E8 950AF1FF      call 004053DC  

004F4947    50               push eax  

004F4948    6A 00            push 0  

004F494A    E8 E92EF1FF      call <jmp.&kernel32.CreateProcessA>  

//运行删除文件的批处理程序  

004F494F    85C0             test eax,eax  

004F4951    74 18            je short 004F496B
```

—————————————————————————————————

```
          ,      \_/  
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /               \\~-.\_ |\\  
       \`\\\\  \_/               \\   ~\\ )          忍把浮名  
   _-~~~-.)  )\_\/;;,.           \\\_  //'/  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \( ,\_.--\\ (     ,;'' /    ~--   /.\_\`\\  
  /~~//'   /`~\\          ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\  


               "     "   "~'  ""

         UnPacKed By :  fly

          2005-04-18 12:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
