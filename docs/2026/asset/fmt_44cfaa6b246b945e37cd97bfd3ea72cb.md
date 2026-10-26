
```asm
5E69AA04  |.  0FB74A 08     movzx ecx, word ptr [edx+8]
5E69AA08  |.  A1 08F6B55E   mov eax, dword ptr [5EB5F608]
5E69AA0D  |.  66:03CE       add cx, si
5E69AA10  |.  81E1 FF3F0000 and ecx, 3FFF
5E69AA16  |.  8B0C88        mov ecx, dword ptr [eax+ecx*4]           ;  获取人物指针
5E69AA19  |.  85C9          test ecx, ecx
5E69AA1B  |.  74 20         je short CXCentau.5E69AA3D
5E69AA1D  |.  8D49 00       lea ecx, dword ptr [ecx]
5E69AA20  |>  8B41 20       /mov eax, dword ptr [ecx+20]
5E69AA23  |.  3950 04       |cmp dword ptr [eax+4], edx
5E69AA26  |.  75 0E         |jnz short CXCentau.5E69AA36
5E69AA28  |.  3971 08       |cmp dword ptr [ecx+8], esi
5E69AA2B  |.  75 09         |jnz short CXCentau.5E69AA36
5E69AA2D  |.  F741 0C 00C00>|test dword ptr [ecx+C], 0C000
5E69AA34  |.  74 19         |je short CXCentau.5E69AA4F
5E69AA36  |>  8B49 2C       |mov ecx, dword ptr [ecx+2C]
5E69AA39  |.  85C9          |test ecx, ecx
5E69AA3B  |.^ 75 E3         \jnz short CXCentau.5E69AA20
5E69AA3D  |>  5E            pop esi
5E69AA3E  |.  33C0          xor eax, eax
5E69AA40  |.  5F            pop edi
5E69AA41  |.  8B4D FC       mov ecx, dword ptr [ebp-4]
5E69AA44  |.  33CD          xor ecx, ebp
5E69AA46  |.  E8 C91A1D00   call CXCentau.5E86C514
5E69AA4B  |.  8BE5          mov esp, ebp
5E69AA4D  |.  5D            pop ebp
5E69AA4E  |.  C3            retn
5E69AA4F  |>  8BC1          mov eax, ecx
5E69AA51  |.  8B4D FC       mov ecx, dword ptr [ebp-4]
5E69AA54  |.  5E            pop esi
5E69AA55  |.  33CD          xor ecx, ebp
5E69AA57  |.  5F            pop edi
5E69AA58  |.  E8 B71A1D00   call CXCentau.5E86C514
5E69AA5D  |.  8BE5          mov esp, ebp
5E69AA5F  |.  5D            pop ebp
5E69AA60  \.  C3            retn
```

信么? 就这些分析了15天..

存表位置找到了, 通过代码遍历

```cpp
void GetCharTable(DWORD dwBase)
{
  if ( dwBase == 0 )
    return;

  int iCount = ReadDword(dwBase + 0x0C);
  dwBase = ReadDword(dwBase + 0x18);

  if ( dwBase != 0 )
  {
    for ( int i = 0; i <= iCount; i ++ )
    {
      DWORD dwAddr = ReadDword(dwBase + i * 4);
      if ( dwAddr != 0 )
      {
        DWORD dwTime = GetTickCount();
        DWORD dwWhile = dwAddr;

        while ( dwWhile != 0 )
        {
          DWORD dwPoint = ReadDword(dwWhile + 4);
          DWORD dwIndex = 0;
          char szText[0x100] = {0};

          if ( dwPoint != 0 )
          {
            DWORD dwName = ReadDword(dwPoint + 0x10);
            if ( dwName != 0 )
              lstrcpyA(szText, (char *)(dwName));

            dwIndex = ReadDword(dwPoint + 0x8);
            DWORD dwValue = ReadDword(dwWhile + 0x0C);
            DbgPrintA("索引:%02X ID:%04X 值:%08X 表名:%s", i, dwIndex, dwValue, szText);
          }

          if ( GetTickCount() - dwTime > 3000 )
          {
            break;
          }

          dwWhile = ReadDword(dwWhile);
        }
      }
    }
  }
}

  DWORD dwMod = (DWORD)GetModuleHandleA("CXCentaur.dll");
  if ( dwMod == 0 )
    return;

  DWORD dwStart = ReadDword(dwMod + g_dwGameBase);
  if ( dwStart != 0 )
  {
    for ( int i = 0; i < 0x3FFF; i ++ )
    {
      DWORD dwBase = ReadDword(dwStart + i * 4);
      if ( dwBase != 0 )
      {
        DWORD dwTimeout = GetTickCount();
        while ( TRUE )
        {
          DWORD dw20 = ReadDword(dwBase + 0x20);
          DWORD dw24 = ReadDword(dwBase + 0x24);
          if ( dw20 != 0 )
          {
            DWORD dw20_4 = ReadDword(dw20 + 4);
            if ( dw20_4 != 0)
            {
              DWORD dw20_4_8 = ReadDword(dw20_4 + 0x8);
              DWORD dw20_4_10 = ReadDword(dw20_4 + 0x10);
              if ( dw20_4_8 == 0x3BE0 )
              {
                DbgPrintA("BASE:%08X 名称指针:%08X 指针:%08X ID:%08X 名称:%s", dwBase, dw20, dw24, dw20_4_8, (char *)(dw20_4_10));
                GetCharTable(ReadDword(dw24 + 6 * 8));
              }
            }
          }

          dwBase = ReadDword(dwBase + 0x2C);
          if ( dwBase == 0 )
            break;
        }
      }
    }
  }
```

人物信息不做过滤 完整的表项如下

```text
索引:00 ID:6800 值:00000000 表名:xl_level
索引:00 ID:B200 值:00000000 表名:jishashenjiang_assigned
索引:01 ID:EF01 值:1FA9240C 表名:achievement
索引:05 ID:9805 值:00000307 表名:gang_contrib
索引:06 ID:8E06 值:0014305E 表名:exp
索引:08 ID:4708 值:00000000 表名:relive
索引:09 ID:9409 值:1FA929EC 表名:enemy
索引:09 ID:7E09 值:1FA9238C 表名:extra_namecard_para
索引:0C ID:D20C 值:00000000 表名:tianyaobuluo_kill
索引:0C ID:140C 值:00000000 表名:plunder_war_data
索引:0E ID:F30E 值:381EFC2C 表名:cd_skills
索引:0E ID:E60E 值:00000000 表名:relive_chances
索引:0F ID:840F 值:00000000 表名:family_task_info
索引:0F ID:3F0F 值:00000000 表名:account_coin
索引:10 ID:2410 值:00000000 表名:newbie_task
索引:10 ID:0E10 值:00000001 表名:vip_level
索引:10 ID:DB10 值:00000000 表名:dadaozhengxiong_schedule
索引:12 ID:3012 值:00000000 表名:wonderland_prepare_to_end
索引:12 ID:D112 值:00000000 表名:douqi
索引:13 ID:6913 值:00000000 表名:arena
索引:14 ID:6914 值:1FA92D8C 表名:condition
索引:14 ID:FA14 值:0000466F 表名:default_cast_skill_id
索引:14 ID:2E14 值:1F756D24 表名:shortcuts
索引:18 ID:6218 值:1FBB94D0 表名:jid
索引:18 ID:F618 值:1FA2FBB0 表名:rid
索引:19 ID:7219 值:00000002 表名:thread
索引:1B ID:681B 值:000000BF 表名:home_point
索引:1B ID:561B 值:00000000 表名:gt_camp_id
索引:1E ID:FD1E 值:1FB4DED4 表名:vip_profit
索引:1E ID:651E 值:1FA9280C 表名:traffic_walk_be_follow
索引:21 ID:8D21 值:00000000 表名:traffic_paused
索引:21 ID:D821 值:49243890 表名:icon_no
索引:22 ID:6422 值:00000000 表名:transparent_value
索引:22 ID:D822 值:00000001 表名:active
索引:24 ID:2E24 值:00000000 表名:round_task_info
索引:25 ID:3825 值:00000000 表名:backup_skills
索引:26 ID:4C26 值:00000000 表名:arena_record
索引:27 ID:0F27 值:00000000 表名:no_guard_monsters
索引:29 ID:7D29 值:00000000 表名:pk_target
索引:2A ID:022A 值:1FA62270 表名:guide
索引:2B ID:7D2B 值:04724BF9 表名:last_keep_distance_lntp_time
索引:2C ID:F72C 值:000000E4 表名:direction
索引:2C ID:9E2C 值:00000000 表名:arena_enroll
索引:2D ID:312D 值:1FA92E4C 表名:current_rank
索引:2E ID:242E 值:0000001B 表名:lntp_latency
索引:2E ID:3E2E 值:00000000 表名:score
索引:30 ID:A830 值:00000000 表名:hide_helmet
索引:30 ID:BC30 值:00000000 表名:arena_camp_id
索引:32 ID:6532 值:1FB10CFC 表名:cd_items_start
索引:33 ID:FB33 值:00000000 表名:quit_gang_time
索引:34 ID:2C34 值:00000000 表名:arena_accumulate_point
索引:35 ID:7135 值:18225A50 表名:action
索引:37 ID:2F37 值:00000000 表名:battlefield_state
索引:38 ID:0138 值:00000000 表名:teleport_point
索引:3B ID:203B 值:00000000 表名:week_popular
索引:3C ID:5B3C 值:1FA925AC 表名:team
索引:3C ID:7B3C 值:00000000 表名:collection_npcs
索引:3D ID:723D 值:00000000 表名:expert
索引:49 ID:6149 值:00000029 表名:level
索引:49 ID:A849 值:00000000 表名:dianfengduijue_processing
索引:4A ID:5D4A 值:1FA62E10 表名:current_talking_npc
索引:4B ID:E24B 值:0000021F 表名:sum_silver_coin
索引:4D ID:D24D 值:00000001 表名:user_camp
索引:4D ID:F34D 值:000002BD 表名:total_douqi
索引:4E ID:474E 值:1FA9324C 表名:seal_break_info
索引:4F ID:664F 值:00000000 表名:newdist_feedback
索引:4F ID:424F 值:00000000 表名:stall_record
索引:4F ID:C84F 值:1FA9935C 表名:my_party_info
索引:51 ID:B451 值:00000000 表名:to_be_delete
索引:51 ID:3E51 值:00000050 表名:gang_station
索引:56 ID:DE56 值:00000000 表名:all_area_triggers_ex
索引:57 ID:EA57 值:311AD274 表名:gang_relations
索引:58 ID:0958 值:00000000 表名:deconnected_time
索引:5D ID:D55D 值:00000000 表名:druglist
索引:5F ID:F35F 值:00000000 表名:tournament_state
索引:60 ID:7460 值:381EFC6C 表名:cd_skills_start
索引:60 ID:5360 值:00000000 表名:halo
索引:60 ID:A460 值:1FA9236C 表名:skills          技能列表
索引:62 ID:4262 值:00000000 表名:money_situ_relive
索引:62 ID:C062 值:00000000 表名:tournament
索引:63 ID:E663 值:000EA632 表名:xw
索引:63 ID:6C63 值:00000000 表名:shortcuts_ex
索引:64 ID:0C64 值:006D6722 表名:last_get_rt_relation_tick
索引:65 ID:E765 值:00000000 表名:emote_group
索引:68 ID:4E68 值:0001BA8E 表名:bind_money        绑定金钱
索引:69 ID:4669 值:49240320 表名:pos          坐标 XYZ
索引:6A ID:216A 值:1FA31000 表名:domain_address
索引:6A ID:B06A 值:1FA61D00 表名:in_territory_war
索引:6A ID:9B6A 值:00000000 表名:battlefield
```
