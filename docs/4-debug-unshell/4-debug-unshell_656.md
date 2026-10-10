

# ExeCryptor 2.x IAT Rebuilder \[SCRIPT\]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-618.htm`](https://www.pediy.com/kssd/pediy08/pediy8-618.htm)

http://www.cracklab.ru/f/index.php?action=vthread&forum=3&topic=6732  
里面注释是俄文的。  
  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> ////////////////////////////////////////////////////////////////
> //
> // EXECryptor 2.x IAT rebuilder by PE\_Kill / REVENGE CREW v 1.9
> // -------------------------------------------------------------
> // Replace my values of iat\_start and iat\_end and run script! 
> //
> // =============================================================
> // - HISTORY -
> // =============================================================
> // \[ v 0.x-1.7 \]
> // Private versions
> //
> // \[ v 1.8 \]
> // Public version
> //
> // \[ v 1.9 \]
> // ! Thanks pavka, for good BugReport
> // \* tracing became some slower, but is more stable.
> // \* little bugfixes in IAT reader.
> // =============================================================
> //
> ////////////////////////////////////////////////////////////////
> 
>  var \_esp
>  var iat\_start
>  var iat\_end
>  var iat\_cur
>  var addr
>  var counter
>  var flag
>  var c\_gpa
>  var temp
>  var gmh
>  var ll
>  var ibase
>  var iend
> 
> //================================================
> // 软桷栲腓玷痼屐 IAT
> //================================================
>  mov iat\_start,00471FF8
>  mov iat\_end,00472EB4
> 
> //================================================
> // 项塍鬣屐 桧纛痨圉棹 ?镳铞羼皴
> //================================================
> 
> //================================================
> // 项塍鬣屐 ImageBase 狃篁铎, ?? 铍� 桧钽溧 礤 
> // 躅麇?溧螯 邈?眍痨嚯 耧铖钺铎
> //================================================
>  mov addr,eip
>  and addr,FFFFF000
>  add addr,1000
> @get\_base:
>  sub addr,1000
>  mov temp,\[addr\]
>  and temp,0000FFFF
>  cmp temp,00005A4D
>  jne @get\_base
> //================================================
> // 铃疱?桤 觚溴疣 ImageSize, 眍 桧钽溧 牮栾蝾?邈?
> // 镱痱栩, 镱铎?狍溴?镥疱耱疣躅怅?..
> //================================================
>  mov ibase,addr
>  mov temp,addr
>  add temp,3C
>  mov temp,\[temp\]
>  add temp,ibase
>  add temp,50
>  mov iend,\[temp\]
>  eval "ImageSize = {iend} ???"
>  msgyn $RESULT
>  cmp $RESULT,1
>  je @run
>  ask "Enter real ImageSize!"
>  mov iend,$RESULT
> @run:
>  add iend,ibase
>  
> 
> //================================================
> // 想篦噱?噤疱襦 老? 觐蝾瘥?桉镱朦珞弪 怵囡镥?
> //================================================
>  gpa "GetModuleHandleA","kernel32.dll"
>  mov gmh,$RESULT
>  gpa "LoadLibraryA","kernel32.dll"
>  mov ll,$RESULT
> //////////////////////////////////////////////////
>  mov \_esp,esp
>  mov iat\_cur,iat\_start
>  sub iat\_cur,4
>  mov c\_gpa,0    // Cryptor GetProcAddress 礤桤忮耱磬
> //=================================================
> // 骤觌 忸耨蜞眍怆屙? IAT
> //=================================================
> @imprec:
>  add iat\_cur,4
>  cmp iat\_cur,iat\_end   // 暑礤?
>  ja @end
>  mov addr,\[iat\_cur\]
>  cmp addr,0    // 朽玟咫栩咫?
>  je @imprec
>  cmp addr,ibase
>  jb @imprec
>  cmp addr,iend
>  ja @imprec
> 
>  mov esp,\_esp    // 骂耨蜞磬怆桠噱?耱尻
>  mov eip,addr    // 埋蜞屐 磬 怵囡镥?
>  mov \[esp\],eip    // 软钽溧 牮栾蝾?麒蜞弪 徉轵 镱 噤疱耋 忸玮疣蜞
>  mov counter,25    // 痒弪麒?蝠囫耔痤怅?
>  mov flag,0    // 丸麇泐 礤 磬殇屙?
> //==================================================
> // 腮屐 觐爨礓?镳钼屦觇 忭篁疱眄彘 IAT
> //==================================================
> @chk\_run:
>  dec counter
>  cmp counter,0    // 篷腓 聍弪麒?钺眢腓腭�
>  je @find\_gpa    // 翌 桎屐 磬 Cryptor gpa
>  sti      // 腻豚屐 ?
>  mov temp,\[eip\]   // 阻蜞屐 徉轵??eip
>  cmp flag,0    // 篷腓 妁?龛麇泐 礤 磬殇屙?
>  ja @chk\_jnz
>  and temp,0000FFFF   // 腮屐 OR EAX,EAX
>  cmp temp,0000C009
>  jne @chk\_run    // 湾?耠邃簋?� 觐爨礓?
>  inc flag    // 袜? 恹耱噔?屐 綦嚆
>  jmp @chk\_run    // 蠕屐 磬 耠邃簋?觐爨礓?
> @chk\_jnz:
>  and temp,0000FFFF   // 腮屐 JNZ xxxxxxxx
>  cmp temp,0000850F
>  jne @chk\_run    // 湾?耠邃簋?� 觐爨礓?
>  mov !ZF,1     // 泥 磬? 泐忸痂?牮栾蝾痼, 黩?忭篁疱眄�� IAT 矬耱?
> 
> @find\_gpa:
>  cmp c\_gpa,0    // 袜?腓 禧 噤疱?Cryptor GetProcAddress?
>  jne @trace    // 篷腓 磬?蝾 桎屐 磬 镱塍麇龛?噤疱襦 老?
>  bphws gmh,"x"    // 软圜?耱噔桁 狃�?磬 GetModuleHandleA
>  bphws ll,"x"    // ?磬 LoadLibraryA
>  ticnd "WORD \[eip\]==453Bh" // 蝠囫耔痼屐, 镱赅 礤 漕殇屐 漕 耩噔礤龛� 觚?
>  bphwc ll    // 耥桁噱?狃�觇 ?老?
>  bphwc gmh
>  cmp eip,gmh    // 帖 磬 GetModuleHandleA?
>  jne @chk\_ll    // 湾?耠邃簋?� 镳钼屦赅...
>  bp \[esp\]    // 软圜?镥疱躅滂?磬 噤疱?忸玮疣蜞
>  run
>  bc eip
>  jmp @find\_gpa    // 耥钼?桎屐 磬 镱桉?Cryptor GetProcAddress
> @chk\_ll:
>  cmp eip,ll    // 帖 磬 GetModuleHandleA?
>  jne @f\_jnz    // 湾?耠邃簋?� 镳钼屦赅...
>  bp \[esp\]    // 软圜?镥疱躅滂?磬 噤疱?忸玮疣蜞
>  run
>  bc eip
>  jmp @find\_gpa    // 耥钼?桎屐 磬 镱桉?Cryptor GetProcAddress
> //=================================================
> // 义镥瘘 棂屐 jnz xxxxxxxx
> //=================================================
> @f\_jnz:
>  sti
>  mov temp,\[eip\] // 659F850F
>  and temp,0000FFFF
>  cmp temp,0000850F
>  jne @f\_jnz
>  mov !ZF,1    // 羼腓 磬? 蝾 ?Z 腩骅?邃桧梓牦, 赅?狍潋?觚 耦怙嚯?
> //==================================================
> // 义镥瘘 棂屐 怛铕簋 镱 聍弪?觐爨礓?ADD EAX,DWORD PTR SS:\[EBP-4\]
> //==================================================
>  mov counter,2    
> @f\_add:
>  sti
>  mov temp,\[eip\] // 68FC4503
>  and temp,00FFFFFF
>  cmp temp,00FC4503
>  jne @f\_add
>  sti
>  dec counter
>  cmp counter,0
>  jne @f\_add 
>  mov c\_gpa,eip    // 义牦?噤疱??狍溴?噤疱耦?Cryptor GetProcAddress
>  sub iat\_cur,4    // 骂玮疣屐 箨噻囹咫?磬 IAT ?镳彐礤?镱腩驽龛?
>  jmp @imprec    // 骂玮疣屐? ?鲨觌 忸耨蜞眍怆屙? 壤?
> 
> //==================================================
> // 项塍麇龛?噤疱襦 老?..
> //==================================================
> @trace:
>  bphws c\_gpa,"x"
>  run
>  bphwc c\_gpa
>  cmp eip,c\_gpa
>  jne @error
>  mov \[iat\_cur\],eax
>  jmp @imprec
> @end:
>  msg "OK!" 
> pause
> ret
> 
> @error:
>  msg "error!!!"
>  pause
> ret
> 
> * * *
> 
> 
> ```

  

 4dd466d0e1e0cc123afbe504da2462ca