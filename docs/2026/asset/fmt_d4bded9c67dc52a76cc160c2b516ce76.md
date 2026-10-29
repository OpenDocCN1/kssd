
| 1 2 3 4 5 6 7 8 9 10 11 12 |
| --- |

```bash
export PATH=/Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/bin:$PATH
export CPLUS_INCLUDE_PATH=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1
export C_INCLUDE_PATH=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/usr/include

# 编译
clang++ -Wall -std=c++11 main.cpp -o main.test

# 报错
Stack dump:
0. Program arguments:
/Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/bin/clang-9 -cc1 -triple x86_64-apple-macosx10.15.0 -Wdeprecated-objc-isa-usage -Werror=deprecated-objc-isa-usage -emit-obj -mrelax-all -disable-free -main-file-name main.cpp -mrelocation-model pic -pic-level 2 -mthread-model posix -mdisable-fp-elim -masm-verbose -munwind-tables -target-sdk-version=10.15.6 -target-cpu penryn -dwarf-column-info -debugger-tuning=lldb -ggnu-pubnames -target-linker-version 609 -resource-dir /Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/lib/clang/9.0.1 -isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk -c-isystem /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/usr/include -cxx-isystem /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1 -stdlib=libc++ -internal-isystem /Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/bin/../include/c++/v1 -internal-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1 -internal-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/local/include -internal-isystem /Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/lib/clang/9.0.1/include -internal-externc-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include -Wall -std=c++11 -fdeprecated-macro -fdebug-compilation-dir /Volumes/Work/CLion/Projects/Test -ferror-limit 19 -fmessage-length 80 -stack-protector 1 -fblocks -fencode-extended-block-signature -fregister-global-dtors-with-atexit -fobjc-runtime=macosx-10.15.0 -fcxx-exceptions -fexceptions -fmax-type-align=16 -fdiagnostics-show-option -fcolor-diagnostics -o /var/folders/vh/l2bql3ns2d5g5w6w2rk1q16m0000gn/T/main-6e5ff9.o -x c++ main.cpp

1. <eof> parser at end of file
2. Per-module optimization passes
3. Running pass 'String encrypt Pass' on module 'main.cpp'.
```

可以看出`clang++`的编译实际上是使用了`clang-9`做的编译，那么修改`CLion`编译配置，尝试调试：

|  |  |
| --- | --- |
| 1 2 3 | `CLion` -> `Run` -> `Edit Configurations`。左边选择`clang`，右边`Executable`选择`clang-9`，`Program arguments`配置： |

```text
-cc1 -triple x86_64-apple-macosx10.15.0 -Wdeprecated-objc-isa-usage -Werror=deprecated-objc-isa-usage -emit-obj -mrelax-all -disable-free -main-file-name main.cpp -mrelocation-model pic -pic-level 2 -mthread-model posix -mdisable-fp-elim -masm-verbose -munwind-tables -target-sdk-version=10.15.6 -target-cpu penryn -dwarf-column-info -debugger-tuning=lldb -ggnu-pubnames -target-linker-version 609 -resource-dir /Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/lib/clang/9.0.1 -isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk -c-isystem /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/usr/include -cxx-isystem /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1 -stdlib=libc++ -internal-isystem /Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/bin/../include/c++/v1 -internal-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1 -internal-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/local/include -internal-isystem /Volumes/Work/CLion/Projects/llvm9.0.1/llvm/cmake-build-debug/lib/clang/9.0.1/include -internal-externc-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include -Wall -std=c++11 -fdeprecated-macro -fdebug-compilation-dir /Volumes/Work/CLion/Projects/Test -ferror-limit 19 -fmessage-length 80 -stack-protector 1 -fblocks -fencode-extended-block-signature -fregister-global-dtors-with-atexit -fobjc-runtime=macosx-10.15.0 -fcxx-exceptions -fexceptions -fmax-type-align=16 -fdiagnostics-show-option -fcolor-diagnostics -o /var/folders/vh/l2bql3ns2d5g5w6w2rk1q16m0000gn/T/main-6e5ff9.o -x c++ /Volumes/Work/CLion/Projects/Test/main.cpp
```

就可以调试了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
