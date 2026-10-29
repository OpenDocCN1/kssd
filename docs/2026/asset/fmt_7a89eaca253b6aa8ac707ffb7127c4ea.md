
```c
// api entry error code definition
enum adp_error_t {
    adp_err_ok = 0, // success
    adp_err_failed, // failed
    adp_err_canceled, // canceled
    adp_err_param, // bad parameter
    adp_err_notfound, // cannot find something
    adp_err_io, // io issue
    adp_err_thread, // thread issue, some api must run at ui thread
    adp_err_oor, // out of range
    adp_err_oom, // out of memory
    adp_err_auth, // license issue
    adp_err_permission, // permission issue
    adp_err_unsupport, // unsupport some action
    adp_err_unimpl, // unimplement some action
    adp_err_softbug, // software bug assertion
    adp_err_continue, // for traverser
    adp_err_break, // for traverser
};
```

```c
// event definition
#define decl_event(n, desc) adp_event_##n
#define decl_event_input(n, input, desc) decl_event(n, desc)
#define decl_event_result(n, result, desc) decl_event(n, desc)
#define decl_event_io(n, input, result, desc) decl_event(n, desc)
enum adp_event_t {
    // event with no Input/Output
    decl_event(loaded, "after loaded this plugin"),
    decl_event(pre_unload, "before unload this plugin"),
    decl_event(main_menu, "user triggered MainMenu/Plugin/ThisPlugin"),
    decl_event(debug_initialized, "tell plugin a new debug session initialized"),
    decl_event(debug_running, "tell plugin the debuggee is running"),
    decl_event(debug_paused, "tell plugin the debugee has bee paused"),
    decl_event(debug_terminated, "tell plugin the debug session has terminated"),
    // event with Input
    // input.ptr is adp_module_t*
    decl_event_input(module_analyzed, ptr, "tell plugin finished analyzing a module"),
    // event for Result
    decl_event_result(version, str_const, "ask this plugin for its sdk version"),
    decl_event_result(menuname, str_const, "ask this plugin for its plugin menu name"),
    // ptr.p0 should be adp's self version string
    // ptr.p1 should be adp's description
    decl_event_result(adpinfo, ptr, "ask this plugin for its self version and description"),
    // event with Input for Result
    // currently nothing
    // ...
    // Tell me, what the extra event do you want ?
};
```

```c
// api definition
struct adp_api_t {
    /* * add by adp v1.0.0 * */
    // get current A64Dbg's version
    const char *( *version)();
    // logger
    void (*log)(const char *msg);
    // logger for status bar
    void (*logStatus)(const char *msg);
    // make main window focus on cpu window
    void (*focusCPU)();
    // make main window focus on log window
    void (*focusLog)();
    // make cpu window goto the specified address
    void (*gotoCPUAdderss)(adpint addr);
    // iterate modules
    void (*travelModule)(adp_error_t (*handler)(const adp_module_t *module));
    // iterate functions
    void (*travelFunc)(const adp_module_t *module, adp_error_t (*handler)(const adp_func_t *func));
    // check whether is debugging
    adpint (*isDebugging)();
    // make dump window goto the specified address, 0,1,2
    adp_error_t (*gotoDumpAddress)(adpint addr, adpint index);
    // get configuration
    adp_error_t (*getIntConfig)(const char *sect, const char *key, adpint *value);
    adp_error_t (*getConfig)(const char *sect, const char *key, char *cfg, adpint cfgsize);
    // set configuration
    adp_error_t (*setIntConfig)(const char *sect, const char *key, adpint value);
    adp_error_t (*setConfig)(const char *sect, const char *key, const char *cfg);
    // ask user to input a string
    adp_error_t (*inputString)(const char *title, char *text, adpint size);
    // ask user to input an integer
    adp_error_t (*inputInteger)(const char *title, adpint *value);
    // ask user to select a path
    adp_error_t (*inputPath)(char *path, adpint size, adpint isdir, adpint isopen);
    // disassemble an arm64 opcode
    adp_error_t (*disassemble)(unsigned opcode, char *asmcode, adpint asmsize);
    // assemble an arm64 asm instruction
    adp_error_t (*assemble)(const char *asmcode, unsigned *opcode);
    // pickup current register value like x0-x29,lr,sp,pc
    adp_error_t (*getRegister)(const char *regname, adpint *regvalue);
    // set register value
    adp_error_t (*setRegister)(const char *regname, adpint regvalue);
    // read memory at addr in the page
    adp_error_t (*readMemory)(adpint addr, adp_bytes_t *buff, adpint *readed);
    // write memory at addr in the page
    adp_error_t (*writeMemory)(adpint addr, const adp_bytes_t *buff, adpint *writed);
    // step one instruction
    adp_error_t (*stepDebugee)(adpint isinto);
    // continue debugee
    adp_error_t (*continueDebugee)();
    // pause debugee
    adp_error_t (*pauseDebugee)();
    // set breakpoint at the specified address
    adp_error_t (*setBreakpoint)(adpint addr, adpint isoneshot, const char *condexpr);
    // unset breakpoint at the specified address
    adp_error_t (*unsetBreakpoint)(adpint addr);
    // set watchpoint at the specified address
    adp_error_t (*setWatchpoint)(adpint addr, adpint size);
    // unset watchpoint at the specified address
    adp_error_t (*unsetWatchpoint)(adpint addr);
    // execute an lldb command
    adp_error_t (*lldbCommand)(const char *cmd);
    adp_error_t (*lldbCommandResult)(const char *cmd, char *result, adpint size);
    // register plugin's command handler, return its id for unregister
    adpint (*registerCommander)(const char *name, bool (*handler)(const char *cmd));
    // unregister command handler, idval is returned by registerCommander
    void (*unregisterCommander)(adpint idval);
    // attach to the pid for selected default platform
    void (*attach)(adpint pid);
    // detach from current debugee
    void (*detach)();
    // ...
    // Tell me, what the extra api do you want ?
};
```
