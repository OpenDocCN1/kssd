
```lua
--返回地址检查
--压入返回地址,调用完毕 jmp回去,返回地址必须是游戏自己调用时的地址
local safe_ret = 0x00000001403AE732
--返回地址,必须用真实的,原因说了,检测地址并且检测字节码...
function luaL_loadbuffer(lua_state, buff, sz, name)
    local asmcode = [[
    push rbp
    push rsp
    sub rsp,0x100
    mov rbp,rsp
    add rbp,0x30

    mov r9, buff_name
    mov r8, buff_size
    mov rdx,pbuff
    mov rcx,lua_state
    mov rax,pfn_luaL_loadbuffer

    mov r15 ,safe_ret --压入返回地址 jmp
    push r15
    jmp rax

    --结束点
    add rsp,0x100

    --代码会从返回地址处jmp回来,正常结束
    pop rsp
    pop rbp
    ret
    ]]

    asmcode = asmcode:gsub("pfn_luaL_loadbuffer", "0x" .. bit64.tohex(pfn_luaL_loadbuffer))
    asmcode = asmcode:gsub("lua_state", "0x" .. bit64.tohex(lua_state))
    asmcode = asmcode:gsub("pbuff", "0x" .. bit64.tohex(ptonumber(buff)))
    asmcode = asmcode:gsub("buff_size", "0x" .. bit64.tohex(sz))
    asmcode = asmcode:gsub("buff_name", "0x" .. bit64.tohex(ptonumber(name)))
    asmcode = asmcode:gsub("safe_ret", "0x" .. bit64.tohex(safe_ret))

    local bcode, code = luaasm.Assemble(asmcode, ptonumber(pasmaddr), "x64")
    ffi.copy(pasmaddr, bcode, #bcode)

    --下面是改写返回地址后的代码,跳回上面的  结束点
    local oldbuf = ffi.new("char[0x20]")
    ffi.copy(oldbuf, ffi.cast("char *", safe_ret), 0x10)--备份返回地址处的代码

    local jmp_code = [[
    mov r15,addr
    jmp r15
    ]]
    jmp_code = jmp_code:gsub("addr", "0x" .. bit64.tohex(ptonumber(pasmaddr) + 0x3d))
    bcode, code = luaasm.Assemble(jmp_code, ptonumber(safe_ret), "x64")
    ffi.copy(ffi.cast("char *", safe_ret + 2), bcode, #bcode) --跳转代码写在返回地址后 + 2 字节处
    local ret = asmcall_c(ptonumber(pasmaddr)) --开始调用

    --调用完毕恢复返回地址的代码
    ffi.copy(ffi.cast("char *", safe_ret), oldbuf, 0x10)
    return ret
end
```

### 发文目的

告诉大家,wow 从调试到注入到调用lua,不需要任何r0技术,不要再用高射炮打蚊子了......

这是2020年1.13.6.36714 作为例子写的,理解了原理套用现在的版本就没问题

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)
