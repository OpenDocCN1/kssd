
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
