
# [分享]Rust内存工具库 hackit

<a href="elink@2b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2V1j5X3N6Q4x3V1k6Z5j5h3y4C8K9i4b7`."><mark class="encrypted">f20K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2V1j5X3N6Q4x3V1k6Z5j5h3y4C8K9i4b7`.</mark></a> 是笔者开发的一个`memory hack`之类的库，中文我不知道咋翻译比较优雅，暂且叫内存工具库
- 主要封装了一些Windows开发常用的功能，比如枚举进程模块、读写内存、Hook、汇编/反汇编、ntdll层的接口封装等，希望能给想用rust做Windows底层开发的朋友提供一些参考
- 开发`hackit`这个库的过程也是我不断学习rust的过程，有些地方的写法可能还不是很规范，有些接口后续可能会变动
- 本文另一个目的是展示使用rust编写代码的便捷性，如果你没用过rust，希望能让你感到新鲜；如果你是老手，希望能抛砖引玉、多多交流
- 本文不介绍rust语言基础的知识，如果想要系统地学习rust，推荐<a href="elink@67eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4Q4x3X3g2J5N6i4y4@1i4K6u0V1L8r3q4F1k6#2)9J5k6h3!0J5k6#2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3u0G2L8$3E0Q4x3V1j5`.">官方教程</a> <a href="elink@9bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6C8j5h3W2K6k6i4u0&6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6@1M7Y4m8D9i4K6u0V1P5X3S2Q4x3X3c8U0L8R3`.`.">中文版</a>
如果想运行下面的示例代码，需要在 `Cargo.toml` 中加入 `hackit` 依赖，使用nightly版本的rust编译器（rust内联汇编需要nightly版本的编译器），我的rustc版本是`rustc 1.52.0-nightly (a8486b64b 2021-02-24)`
```toml
[dependencies]
hackit = {git = 'https://gitee.com/udbg/hackit', features = ['std', 'csutil']}
```

## 枚举进程、模块

枚举进程
```rust
use hackit::\*;
for ps in enum\_process() {
println!("{} {}", ps.pid(), ps.name());
}
```
利用rust的迭代器trait，可以很方便地过滤出自己想要枚举的进程,
更多迭代器方法可参考官方文档 <mark class="encrypted">ccfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4Q4x3X3g2J5N6i4y4@1i4K6u0V1L8r3q4F1k6#2)9J5k6h3!0J5k6#2)9J5c8X3y4G2M7X3g2Q4x3V1k6A6N6r3g2J5i4K6u0r3N6s2u0S2K9i4c8Q4x3X3g2u0N6r3g2J5j5i4c8G2M7W2)9J5k6h3S2@1L8h3H3`.</mark>
```rust
for ps in enum\_process().filter(|p| p.name().eq\_ignore\_ascii\_case("svchost.exe")) {
println!("{} {}", ps.pid(), ps.name());
}
```
通过进程名过滤进程，被单独封装成了一个函数 `enum\_process\_filter\_name(name: &str)`
获得第一个名为 explorer.exe 的进程信息
```rust
let ps = enum\_process\_filter\_name("explorer.exe").next().unwrap();
println!("{} {}", ps.pid(), ps.name());
```
遍历模块：枚举 explorer 进程中的模块
```rust
for m in enum\_module(ps.pid()) {
println!("{:x} {:x} {}", m.base(), m.size(), m.path());
}
```

## 进程内存读写
```rust
// 打开进程对象，第二个参数为访问权限，默认为 PROCESS\_ALL\_ACCESS
let p = Process::open(ps.pid(), None).expect("open explorer");
// 定位到模块基址
let base = p.enum\_module().next().unwrap().base();
// 读取内存
let buf = p.read\_bytes(base, 2);
assert\_eq!(buf, b"MZ");
// 泛型读取，泛型参数可以是 u16 u32 float double ...
let word = p.read\_value::<u16>(base).unwrap();
println!("read u16: 0x{:x}", word);
// 读字符串
let s = p.read\_cstring(base, 2);
println!("read string: {:?}", s);
```
更多读内存的方法参考 `trait ReadMemUtil` <mark class="encrypted">5e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2V1j5X3N6Q4x3V1k6Z5j5h3y4C8K9i4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3L8h3g2E0i4K6u0W2M7Y4y4Q4x3U0y4x3x3U0p5`.</mark>

## 部分ntapi接口的封装
```rust
// 枚举驱动模块
for m in system\_module\_list().unwrap() {
println!("{:p} {:x} {}", m.ImageBase, m.ImageSize, m.full\_path\_str());
}
// 枚举 explorer 的进程线程信息
for pi in system\_process\_information().unwrap().filter(|pi| pi.UniqueProcessId as u32 == p.pid) {
println!("pid: {} image: {} handle-count: {}", pi.UniqueProcessId as u32, pi.ImageName.to\_string(), pi.HandleCount);
for ti in pi.threads() {
println!(
" tid: {} priority: {} wait-time: {} entry: {:p}",
ti.ClientId.UniqueThread as u32,
ti.Priority,
ti.WaitTime,
ti.StartAddress,
);
}
}
```

## 字符串编码处理

rust的String是utf8编码，如果需要调用一些Windows原生接口，需要转Unicode字符串
```rust
// utf8转Unicode
let abc = "abc";
unsafe {
use core::ptr::\*;
use winapi::um::winuser::MessageBoxW;
MessageBoxW(null\_mut(), abc.to\_unicode\_with\_null().as\_ptr(), null\_mut(), 0);
}
// Unicdoe转utf8
let wstr = abc.to\_unicode();
assert\_eq!(wstr.to\_utf8(), "abc");
// Unicode转ANSI编码
assert\_eq!(wstr.to\_ansi(), b"abc");
```

## Hook功能

直接贴了测试用例，Hook库是自己实现的，理论上是可以Hook任意地址的（有跳转指令的除外，处理jmp和call的逻辑还有点问题）
```rust
#[cfg(test)]
mod hook\_test {
use super::\*;
use std::println;
use winapi::um::winuser::\*;
use winapi::um::libloaderapi::\*;
const MAGIC\_VALUE: u64 = 1001234;
#[test]
fn inline\_hook() {
let tp = this\_process();
tp.sym\_init(None, true).ok();
type FnRtlQueryPerformanceCounter = extern "system" fn(&mut u64) -> usize;
let m = unsafe { GetProcAddress(GetModuleHandleA(b"ntdll\0".as\_ptr() as \*const i8), b"RtlQueryPerformanceCounter\0".as\_ptr() as \*const i8) };
assert!(!m.is\_null());
HookManager::instance().inline\_hook(m as usize, |args: &mut HookArgs| unsafe {
let RtlQueryPerformanceCounter = transmute::<\_, FnRtlQueryPerformanceCounter>(args.trampoline());
let regs = &mut args.regs;
let arg1 = regs.arg(1);
\*regs.ax() = RtlQueryPerformanceCounter(transmute(arg1));
\*(arg1 as \*mut u64) = MAGIC\_VALUE;
args.reject = Some(if size\_of::<usize>() == 8 { 0 } else { 1 });
}, true).expect("RtlQueryPerformanceCounter");
let mut counter: u64 = 0;
unsafe {
let RtlQueryPerformanceCounter: FnRtlQueryPerformanceCounter = transmute(m);
RtlQueryPerformanceCounter(transmute(&mut counter));
assert\_eq!(MAGIC\_VALUE, counter);
}
let m = unsafe { GetProcAddress(LoadLibraryA(b"user32\0".as\_ptr() as \*const i8), b"MessageBoxA\0".as\_ptr() as \*const i8) };
assert!(!m.is\_null());
static mut CHECK\_MSG: u64 = 0;
unsafe extern "C" fn callback(args: &mut HookArgs) {
// let msg: extern "system" fn(usize, \*const u8, \*const u8, u32) = transmute(args.ret);
// msg(0, b"Hooked\0".as\_ptr(), "\0".as\_ptr(), 0);
CHECK\_MSG = MAGIC\_VALUE;
args.reject = Some(if cfg!(target\_arch = "x86\_64") { 0 } else { 4 });
}
HookManager::instance().inline\_hook(m as usize, callback as HookCallbackFn, true).expect("MessageBoxA");
unsafe {
MessageBoxA(std::ptr::null\_mut(), b"error\0".as\_ptr() as \*const i8, b"\0".as\_ptr() as \*const i8, 0);
assert\_eq!(MAGIC\_VALUE, CHECK\_MSG);
}
}
#[test]
fn table\_hook() {
static mut CHECK\_HOOK: u64 = 0;
unsafe {
let m = GetProcAddress(LoadLibraryA(b"kernel32\0".as\_ptr() as \*const i8), b"OutputDebugStringW\0".as\_ptr() as \*const i8);
let output\_debug\_string: extern "win64" fn(LPCWSTR) = transmute(m);
let address: usize = transmute(&output\_debug\_string);
HookManager::instance().table\_hook(address, |\_arg: &mut HookArgs| {
println!("TableHook success");
CHECK\_HOOK = MAGIC\_VALUE;
}, true).expect("table hook");
let s = "OutputDebugString".to\_wide();
output\_debug\_string(s.as\_ptr());
assert\_eq!(MAGIC\_VALUE, CHECK\_HOOK);
}
}
}
```

## 内联汇编、ShellCode

主要是实现了`shellcode!`这个宏，通过这个宏直接编写汇编代码，并能返回一个指向这段汇编的`&[u8]`(字节数组引用)
<mark class="encrypted">33bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2V1j5X3N6Q4x3V1k6Z5j5h3y4C8K9i4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3N6i4c8A6L8q4)9J5k6i4u0K6i4K6t1K6e0o6V1#2</mark>
用法参考Hook库的实现 <mark class="encrypted">00eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2V1j5X3N6Q4x3V1k6Z5j5h3y4C8K9i4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3N6$3W2F1i4K6u0r3K9r3!0G2K9#2)9J5k6i4u0K6i4K6t1K6e0o6p5@1x3H3`.`.</mark>
```rust
use hackit::util::\*;
fn create\_hook\_handler() -> \*const u8 {
shellcode!(
"push rsp" // +80
"push rax" // +78
"push rcx" // +70
"push rdx" // +68
"push rbx" // +60
"push rbp" // +58
"push rsi" // +50
"push rdi" // +48
"push r8" // +40
"push r9" // +38
"push r10" // +30
"push r11" // +28
"push r12" // +20
"push r13" // +18
"push r14" // +10
"push r15" // +8
"pushfq" // +0
"add qword ptr [rsp+0x80], 8"
"mov rcx, [rsp+0x88]"
"mov rdx, rsp"
"sub rsp, 0x18"
"call hook\_handler"
"add rsp, 0x18"
"popfq"
"pop r15"
"pop r14"
"pop r13"
"pop r12"
"pop r11"
"pop r10"
"pop r9"
"pop r8"
"pop rdi"
"pop rsi"
"pop rbp"
"pop rbx"
"pop rdx"
"pop rcx"
"pop rax"
"pop rsp"
"ret"
).as\_ptr()
}
```

## 二进制搜索

主要是实现了`sig!`这个宏，可以直接写BinaryPattern，搜索算法使用了最简单的顺序匹配
