
```c
                          //OutputDebugStringA("HELLP");

                                HIMC myhIMC = ImmGetContext(hwnd);

                                SendSomething\_himc(myhIMC,ImeStr);

                          Game\_dx.DATA2->myImeMessage.IsSendIme=FALSE;

                          }

                  }

          }

                  if(uMsg==WM\_USER+2345)//检测发送过来的绑定检测消息 防止2次绑定失效

                  {

                          //OutputDebugStringA("2345");

                          Hook\_dx();

                  }

                   if (uMsg==WM\_USER+6789)//内存CALL

                   {

                   }

                   if (uMsg==WM\_USER+67867)//测试出售功能 可删除

                   {

                          // Do\_Same();

                           //OutputDebugStringA("chushou");

                   }

                   if (uMsg==WM\_USER+6777)

                   {

                        /\*   RenWu\_X\_Y();

                           return 333;\*/

                   }

                   if(    uMsg==WM\_USER+65432  )

                   { //OutputDebugStringA("65432");

                           if(wParam==1)//得到怪物数量

                           {

                                   Game\_dx.DATA2->GuaiwuShuliang=Gamefu->Get\_GuaiWuShuLiang(0);

                                   return Game\_dx.DATA2->GuaiwuShuliang;

                           }

                           if(wParam==3)//得到物品数量

                           {

                                   Game\_dx.DATA2->WUPUN\_WULIANG=Gamefu->Get\_WuPinShuLiang(0);

                                   return Game\_dx.DATA2->WUPUN\_WULIANG;

                           }

                           if(wParam==4)//人物瞬移

                           {

                                   //OutputDebugStringA("SHUN YI");

                                   Gamefu->Xi\_Men(lParam);

                           }

                           if(wParam==5)//人物瞬移

                           {

                                   //OutputDebugStringA("SHUN YI");

                                   Gamefu->Xi\_Men\_Befor(lParam);

                           }

                           if(wParam==6)//倍功

                           {

                                  // MyCall(GetDnfPid(),RenwuJizhi,\_ShangDianJiZhi,20035,2);

                                 return Gamefu->SetJiami\_Hot\_BeiGong(lParam);

                           }

                /\*           if(wParam==7)//修改力量

                           {

                                   return Gamefu->SetJiami\_Hot\_BeiGong\_Liliang(lParam);

                           }\*/

                                if(Game\_dx.DATA2->Isadd\_Liliang==TRUE)

                                {

                                        Gamefu->SetJiami\_Hot\_BeiGong\_Liliang(Game\_dx.DATA2->liliang);

                                        Game\_dx.DATA2->Isadd\_Liliang=FALSE;

                                }

                                if (Game\_dx.DATA2->IsRetTili==TRUE)

                                {

                                        Game\_dx.DATA2->Rettili=Gamefu->GetCurTili();

                                        Game\_dx.DATA2->IsRetTili=FALSE;

                                }

                                if (Game\_dx.DATA2->Tili!=0)

                                {

                                 Gamefu->Add\_TiLi(Game\_dx.DATA2->Tili);

                                 Game\_dx.DATA2->Tili=0;

                                }

                                if (Game\_dx.DATA2->Lv!=0)

                                {

                                 Gamefu->Add\_Lv(Game\_dx.DATA2->Lv);

                                 Game\_dx.DATA2->Lv=0;

                                }/\*

                           if(wParam==8)//查血

                           {

                                 return Gamefu->Add\_Hp();

                           }

                           if(wParam==9)//加体力

                           {

                                 return Gamefu->Add\_TiLi(lParam);

                           }

                           if(wParam==10)//加等级

                           {

                                 return Gamefu->Add\_Lv(lParam);

                           }

                           if(wParam==11)//查体力

                           {

                                 return Gamefu->GetCurTili();

                           }\*/

                                return 0;

                   }

                   if (uMsg==WM\_USER+89898 )

                   {

                           return GetMen(wParam);

                   }

                   if(uMsg==WM\_USER+75741)

                   {

                           if(wParam==1)

                           {

                                   if (lParam==0)//得到物品背吧列表

                                   {

                                           if(Game\_dx.DATA2!=NULL)

                                           {

                                                   Gamefu->GetBeiBaoToArray(\_BeiBaoJizhi,&Game\_dx.DATA2->GameBeiBao);

                                                   return 1;

                                           }

                                   }

                                   if (lParam==1)

                                   {

                                           if(Game\_dx.DATA2!=NULL)

                                           {

                                                   Gamefu->GetCurZhuangBeiToArray(&Game\_dx.DATA2->curzb);

                                                   return 1;

                                           }

                                   }

                                   // Game\_dx.Get\_SendData();

                           }

                           if (wParam==2)

                           {

                                   if (lParam==0)//得到角色等级和名字疲劳负重

                                   {

                                            Gamefu->GetRenwu\_Lv\_Name(RenwuJizhi, Game\_dx.DATA2->Name\_Lv);

                                                DWORD CurPilao=Gamefu->GetPiLao();

                                                Game\_dx.DATA2->Name\_Lv.Pilao=CurPilao;

                                                DWORD AllFuzhong=Gamefu->GetAllFuZhong();

                                                Game\_dx.DATA2->Name\_Lv.MaxFuzhong=AllFuzhong;

                                                DWORD  CurFuzhong=Gamefu->GetFuZhong();

                                                Game\_dx.DATA2->Name\_Lv.CurFuzhong=CurFuzhong;

                                                  Game\_dx.SendTo\_Game();

                                                return 1;

                                   }

                                   if (lParam==1)//获取金币数量

                                   {

                                           return Gamefu->GetJinBi(\_BeiBaoJizhi);

                                   }

                           }

                   }

        return CallWindowProcA((WNDPROC) OldWindowProc,hwnd, uMsg, wParam, lParam );

}

ULONG\_PTR Old\_GetForegroundWindow\_add5=0;

\_\_declspec(naked) HWND \_\_stdcall TMP\_MyGetForegroundWindow()

{

        \_\_asm{

                nop

                        nop

                        nop

                        nop

                        nop

                        nop

                        jmp Old\_GetForegroundWindow\_add5

        }

}

HWND \_\_stdcall MyGetForegroundWindow()

{

        if (is\_hook\_user==FALSE )

        {

                if (Game\_dx.IsCur()==TRUE)

                {

                        Hook\_dx();

                        HookPrsent();

                        is\_hook\_user=TRUE;

                }

        }

        HWND wnd1=TMP\_MyGetForegroundWindow();
```
