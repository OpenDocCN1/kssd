
``` 
return
nf->call((const uintptr_t*)((va_list)_ADDRESSOF(name) + _INTSIZEOF(name)));

}
uintptr_t call(uintptr_t func_address, const uintptr_t* param_list, size_t param_list_size)
{
uintptr_t retval;
uintptr_t esp_ptr;
size_t param_size = param_list_size * sizeof uintptr_t;
_asm
{
sub esp, param_size;
mov esp_ptr, esp;
}
memcpy((void*)esp_ptr, param_list, param_size);
_asm
{
call [func_address];
mov esp, esp_ptr;
add esp, param_size;
mov retval, eax;
}
return retval;
}
```

总结一下，修改存储的玩家，全部为自己即本地玩家，修改游戏结果为字符串“1”。

在提交漏洞和沟通以前我并不清楚，一个“理应”有投票机制的积分系统怎么会出现单人操控游戏结果的漏洞，直到我从他们主管那里听到，说他们开发在写服务端的时候把我一个人的结果算成了100个人的结果，也就是说如果没人篡改，那么ok，有人篡改的话，篡改的玩家结果x100，假设有10个人进行游戏，9个人的结果是正确的，1个人结果是篡改过的，但是那一个人被当成了100个人，所以100>9，由此得出篡改数据的玩家的数据为最终结果，真NMSB。。。。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
