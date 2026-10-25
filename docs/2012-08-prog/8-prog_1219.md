
# writeprocessmemory 学习的结果

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-575.htm`](https://www.pediy.com/kssd/pediy09/pediy09-575.htm)

> 为了学几个 API 的用法，结果弄出了个修改器，大牛们不要笑话就是了。`![尴尬](img/4ec4a010bb6e6d79731981cd94fbb640.jpg)`

> ```pascal
> implementation
> 
> uses
> TestHookKey_Unit;
> 
> {$R *.dfm}
> 
> procedure TForm2.openPr;  // 打开进程...
> begin
>   if ComboboxEX1.itemindex = -1 then exit;
>   List := TStringList.Create;
>   FProcessEntry32 := GetProcessID(List, ComboboxEX1.text);
>   if FProcessEntry32.th32ProcessID=0 then exit;
>   ProcessID := FProcessEntry32.th32ProcessID;
>   ProcessHandle := OpenProcess(PROCESS_ALL_ACCESS, false, ProcessID);
>   List.free;
> end;
> 
> 
> procedure TForm2.openhero; // 打开英雄进程
> begin
>   List := TStringList.Create;
>   FProcessEntry32:= GetProcessID(List, 'Heroes3.exe');
>   if FProcessEntry32.th32ProcessID=0 then exit;
>   ProcessID := FProcessEntry32.th32ProcessID;
>   ProcessHandle := OpenProcess(PROCESS_ALL_ACCESS, false, ProcessID);
>   List.free;
> end;
> 
> // 找玩家的基础地址，成功返回地址，否则返回 0
> function TForm2.findaddr(Hex: Cardinal): Cardinal;
> var
>  pdd:^dword;
>  hg:dword;
> begin
>   result:=0;
>   hg:=hex;
>   nSize:=16;
>   pdd:= AllocMem(nSize);
>   while hg < $02000000 do
>   begin
>     try
>      ReadProcessMemory(ProcessHandle, Pointer(hg), pdd, nSize,
>        lpNumberOfBytes);
>    if (pdd^ = 8) then
>       begin
>        inc(pdd);
>        if (pdd^ = $01000100) then
>        begin
>           inc(pdd);
>           if (pdd^ = $FFEEFFEE) then
>           begin
>               inc(pdd);
>               if (pdd^ = 0) then
>                begin
>                  result:= hg;
>                  exit;
>                end
>               else  result:=0;
>           end;
>        end;
>       end;
>     hg:=hg+$10000;
>     except
>     hg:=hg+$10000;
>     end;
>   end;
> end;
> 
> procedure TForm2.closePr; // 关闭进程
> begin
>   CloseHandle(ProcessHandle);
> end;
> 
> 
> // 传入英雄的地址，返回英雄的名字
> function TForm2.getname(Hex: Cardinal): string;
> var
>   mst:^string;
>   haddr:dword;
> begin
>   haddr:=hex;
>   nSize:=12;
>   begin
>     mst:=AllocMem(13);
>     if(not ReadProcessMemory(ProcessHandle, Pointer(haddr), mst, nSize, lpNumberOfBytes))then
>     begin
>        showmessage('读英雄的名字出错，可能是指定的地址不存在.');
>        exit;
>     end;
>     asm
>      mov eax,result
>      mov edx,mst
>      mov [eax],edx
>     end;
>   end;
> end;
> 
> 
> // 传入玩家首地址，返回所有玩家的英雄
> function TForm2.getallname(Hex: Cardinal):Boolean;
> var
>   c,i,j,by,byt:byte;
>   pb:^byte;
>   haddr,xuhao:dword;
>   pdd:^dword;
> begin
>   result:=false; //
>   c:=0;
>   for j:=0 to 7 do
>   begin
>     nSize:=2;
>     pb:= AllocMem(4);
>     if(not ReadProcessMemory(ProcessHandle, Pointer(hex), pb, nSize, lpNumberOfBytes))then
>     begin
>        closePr;
>        exit;
>     end;
>   by:=pb^;
>   inc(pb);
>   if pb^ > 0 then
>    begin
>      haddr:=hex+8;
>      byt:=pb^;
>      for i := 1 to byt do
>      begin
>         nSize:=4;
>         pdd:= AllocMem(nSize);
>         ReadProcessMemory(ProcessHandle, Pointer(haddr), pdd, nSize, lpNumberOfBytes);
>         xuhao:=pdd^;
>         hercolor[c]:=by;
>         herono[c]:=xuhao;
>         hinfo.currhero:=xuhao*$492+$21620+hinfo.basehero;
>         hername[c]:=getname(hinfo.currhero+$23);
>         haddr:=haddr+4;
>         c:=c+1;
>      end;
>      result:=true;
>    end;
>    hex:=hex+360;
>   end;
>   herono[69]:=c;
> end;
> 
> 
> function TForm2.GetProcessID(var List: TStringList; FileName: string = ''): TProcessEntry32;
> var
>   Ret: BOOL;
>   s: string;
> begin
>   FSnapshotHandle := CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
>   FProcessEntry32.dwSize := Sizeof(FProcessEntry32);
>   Ret := Process32First(FSnapshotHandle, FProcessEntry32);
>   while Ret do
>   begin
>     s := ExtractFileName(FProcessEntry32.szExeFile);
>     if (FileName = '') then
>     begin
>       List.Add(Pchar(s));
>     end
>     else if (AnsiCompareText(Trim(s),Trim(FileName))=0) and (FileName <> '') then
>     begin
>       List.Add(Pchar(s));
>       result := FProcessEntry32;
>       break;
>     end;
>     Ret := Process32Next(FSnapshotHandle, FProcessEntry32);
>   end;
>   CloseHandle(FSnapshotHandle);
> end;
> 
> procedure TForm2.suiButton5Click(Sender: TObject);
> var
>    haddr:dword;
> begin
>   haddr:=hinfo.baseplayer+$9C+hinfo.player*$168;
>   nSize:=sizeof(zhiyuan);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @zhiyuan, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写资源出错!');
>     end;
>   ///////////////////////////写魔法值
>   haddr:=hinfo.currhero+$18;
>   nSize:=sizeof(mofazi);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @mofazi, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写魔法值出错!');
>     end;
>   //////////////////////// 写移动点数
>   haddr:=hinfo.currhero+$49;
>   nSize:=sizeof(yidong);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @yidong, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写移动点数出错!');
>     end;
>   //////////////////////////////// 写经验值
>   haddr:=hinfo.currhero+$51;
>   nSize:=sizeof(jinyanzi);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @jinyanzi, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写经验值出错!');
>     end;
>   //////////////////////////////
>   ////////////////////////  写兵种
>   haddr:=hinfo.currhero+$91;
>   nSize:=sizeof(bingzh);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @bingzh, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写兵种数据出错!');
>     end;
>   //////////////////////// 写兵力
>   haddr:=hinfo.currhero+$AD;
>   nSize:=sizeof(bingli);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @bingli, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写兵力数据出错!');
>     end;
>   ////////////////////////  写辅助技能
>   haddr:=hinfo.currhero+$C9;
>   nSize:=sizeof(fcjineng);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @fcjineng, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写辅助技能出错!');
>     end;
>   ////////////////////////  写辅助技能的显示
>   haddr:=hinfo.currhero+$E5;
>   nSize:=sizeof(jnmanban);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @jnmanban, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写辅助技能出错!');
>     end;
>   ////////////////////////
> 
>   haddr:=hinfo.currhero+$1d4;
>   nSize:=sizeof(baowu);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @baowu, nSize, lpNumberOfBytes))
>         or(nSize<>lpNumberOfBytes) then
>     begin
>        showmessage('写宝物数据出错!');
>     end;
>   /////////////////// 写魔法
>   haddr:=hinfo.currhero+$430;
>   nSize:=sizeof(mofa);
>   if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @mofa, nSize, lpNumberOfBytes))
> ```



```
 or(nSize<>lpNumberOfBytes) then  
    begin  
       showmessage('写魔法数据出错!');  
    end;  
 ////////////////////////// 写基本技能  
 haddr:=hinfo.currhero+$476;  
 nSize:=sizeof(jbjineng);  
 if(not WriteProcessMemory(ProcessHandle, Pointer(haddr), @jbjineng, nSize, lpNumberOfBytes))  
       or(nSize<>lpNumberOfBytes) then  
    begin  
       showmessage('写基本技能出错!');  
    end;  
 //////////////////////////  
 closepr;  
 Form2.Hide;  
 end;  

 procedure TForm2.suiButton6Click(Sender: TObject);  
 begin  
  closepr;  
  Application.Terminate;  
 end;  

 procedure TForm2.FormShow(Sender: TObject);  
 var  
  laddr : dword;  
 begin  
  hinfo.cyteaddr:=0;  
  hinfo.basehero:=0;  
  hinfo.currhero:=0;  
  hinfo.baseplayer:=0;  
  PageControl1.ActivePage:=TabSheet1;  
  openPr;  
  if FProcessEntry32.th32ProcessID=0 then  
   begin  
   showmessage('你没有启动游戏?'+#10#13+'请你启动游戏后，点“读数据”按钮。');  
   exit;  
   end;  
  laddr:=findaddr($01500000);  
  if laddr = 0 then  
  begin  
   showmessage('你没有打开地图?'+#10#13+'请你先打开地图，再点“读数据”按钮。');  
   exit;  
   end;  
  hinfo.basehero:=laddr+$68;  
  hinfo.baseplayer:=laddr+$20B38; //第一位玩家 $360 长度  
  if getallname(hinfo.baseplayer)= false then  
   begin  
   showmessage('没有找到活动英雄！你没有打开地图?'+#10#13+'请你先打开地图，再点“读数据”按钮。');  
   ProcessID:=0;    
   end;  
 end; 
```

`4dd466d0e1e0cc123afbe504da2462ca`
