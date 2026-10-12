
#### 4、BIOS 修改工具

例如 `CBROM`，我使用 `v2.08`，`v2.07` 和 `1.24`。以及 `MODBIN`，有两种：`modbin6` for `award bios ver. 6` 和 `modbin 4.50.xx` for `award bios ver. 4.5xPGNM`。使用这些工具更容易查看 BIOS 组件。可从 `www.biosmods.com` 下载。

#### 5、芯片集数据表

这取决于你要解剖的主板 BIOS 代码。`www.com.by` 上有一部分 pdf 格式的数据表。我解剖的主板是 `VIA693A-596B`，我当然有这个数据表。

#### 6、Intel Software Developer Manual Volume 1, 2 and 3

BIOS 有时使用一些外来指令集。另外有些很难记住的数据结构需要查询，如 `GDT`、`IDT` 等。

## 5.2. IDA Pro 技术介绍

本小节介绍使用 `IDA Pro`。如果抓住了这些概念，你可以方便地使用 `IDA pro`。

### 5.2.1. IDA Pro 介绍

逆向代码工程通过分析软件的可执行文件来实现对软件所使用算法的理解。在大多数情况下，软件仅发布它的可执行文件而没有源代码。BIOS 也同样如此，我们可获得的仅仅是执行代码。逆向代码工程在以下工具的帮助下实现：调试器，反汇编工具，二进制文件编辑器即二进制编辑器，`ICE`（）等。我们在本小节中仅讨论反汇编工具，例如 `IDA Pro` 反汇编工具。

`IDA Pro` 是一款强大的反汇编工具。它支持插件和脚本组件支持 50 种以上的处理器结构。但功能强大的工具一般都有缺陷，就是难以掌握使用，`IDA Pro` 也不例外。

`IDA Pro` 有多个版本：免费版、标准版和高级版。最新的免费版为 `IDA Pro version 4.3 (AFAIK)`，可在 `http://www.dirfile.com/ida\_pro\_freeware\_version.htm` 下载。

There are several editions of `IDA Pro`: freeware edition, standard edition and advanced edition. The latest freeware edition is `IDA Pro version 4.3 (AFAIK)` and it available for download at `http://www.dirfile.com/ida\_pro\_freeware\_version.htm`. It the most limited of all `IDA Pro` version. It only supports `x86` processor and doesn't come with plugin feature, but it comes at no cost, that's why it's presented here. Fortunately, it still comes with scripting feature. The standard and advanced editions of `IDA Pro 4.3` of course differ from this freeware edition, they come with support for plugin and support for much more processor architecture. We are going to learn how to use the scripting feature in the next section.

Now, let start to use `IDA Pro` freeware version to open a BIOS binary file. First, `IDA Pro` freeware version has to be installed. After the installation finished, one special step must be carried-out to prevent unwanted bug when this version of `IDA Pro` opens up a BIOS file with `*.rom` extension. To do so, one must edit the `IDA Pro` configuration file that located in the root directory of the `IDA Pro` installation directory. The name of the file is `ida.cfg`. Open this file by using any text editor (such as `notepad`) and look for the following lines:

```
DEFAULT_PROCESSOR = {
/* Extension    Processor */
  "com" :       "8086"                    // IDA will try the specified
  "exe" :       ""                        // extensions if no extension is
  "dll" :       ""                        // given.
  "drv" :       ""
  "sys" :       ""
  "bin" :       ""  // Empty processor means the default processor
  "ovl" :       ""
  "ovr" :       ""
  "ov?" :       ""
  "nlm" :       ""
  "lan" :       ""
  "dsk" :       ""
  "obj" :       ""
  "prc" :       "68000"                 // PalmPilot programs
  "axf" :       "arm710a"
  "h68" :       "68000"                 // MC68000 for *.H68 files
  "i51" :       "8051"                  // i8051   for *.I51 files
  "sav" :       "pdp11"                 // PDP-11  for *.SAV files
  "rom" :       "z80"                   // Z80     for *.ROM files
  "cla*":       "java"
  "s19":        "6811"
  "o":          ""
  "*":          ""                      // Default processor
}
```

Notice the line: `"rom" : "z80" // Z80 for *.ROM files`

This line must be removed or just replace the `"z80"` with `""` in this line to disable the automatic request to load `z80` processor module in `IDA Pro` upon opening a `*.rom` file. The bug occurred if the `*.rom` file is opened while this line is not changed ince freeware `IDA Pro` doesn't come with `z80` processor module. Thus, opening `*.rom` file by default will terminate `IDA Pro`. Some motherboard BIOS files comes with `*.rom` extension by default, even though it's very clear that it won't be executed in `z80` processor. Fixing this bug will ensure that we will be able to open motherboard BIOS file with `*.rom` extension flawlessly. Note that the steps needed to remove other file-extension to processor-type "mapping" in this version of `IDA Pro` is similar to the `z80` processor that is just described.

Now let's proceed to open a sample BIOS file. This BIOS file is `da8r9025.rom`, BIOS file for `Supermicro H8DAR-8 (OEM Only)` motherboard. This motherboard used `AMD-8131 HyperTransport PCI-X Tunnel` chip and `AMD-8111 HyperTransport I/O Hub` chip. The dialog box below will be displayed when you start `IDA Pro` freeware version `4.3`.

Just click `OK` to proceed. Then the next dialog box shown below will be displayed.

In this dialog box, you can try one of the three options, but we will just click on the `Go` button. This will start `IDA Pro` with empty workspace as shown below

Then locate and drag the file to be disassembled to the `IDA Pro` window (as shown above). In this case, `IDA Pro` will show the following dialog box.

In this dialog box, we will select `Intel 80x86 processors: athlon` as the Processor type in the drop down list box. Then click on the `Set` button to activate the new processor selection. Let the other option as it is. Code relocation will be carried out by using `IDA Pro` scripts in later section, then click `OK`. `IDA Pro` then shows the following dialog box.

This dialog box asks us to choose the default operating-mode of the `x86` compatible processor during the disassembling process. `AMD64 Architecture Programmer Manual Volume 2: System Programming, February 2005` in section `14.1.5` page `417` states that:

"After a `RESET#` or `INIT`, the processor is operating in `16-bit` real mode."

In addition, `IA-32 Intel Architecture Software Developer Manual Volume 3: System Programming Guide 2004` section `9.1.1` states that:

"Table `9-1` shows the state of the flags and other registers following power-up for the `Pentium 4`, `Intel Xeon`, `P6 family`, and `Pentium` processors. The state of control register `CR0` is `60000010H` (see Figure `9-1`), which places the processor is in real-address mode with paging disabled."

Thus, we can conclude that any `x86` compatible processors start their execution in `16-bit` real mode just after power-up and we have to choose `16-bit` mode in this dialog box. It accomplished by clicking `No` in the dialog box. Then the following dialog box pops up.

This dialog box told us that `IDA Pro` can decide where the entry-point located. We have to locate it ourselves later. Just click `OK` to continue to the main window for the disassembly process.

Up to this point we are able to open the binary file within `IDA Pro`. This is not a trivial task for people new to `IDA Pro`. That's why it's presented in a step-by-step fashion. However, the output in the workspace is not yet usable. The next step is learning the scripting facility that `IDA Pro` provides to make sense about the disassembly database that `IDA Pro` generates.

### 5.2.2. IDA Pro Scripting And Key Bindings

Now we will proceed to try to decipher `IDA Pro` disassembly database shown in the previous sub-section with the help of the scripting facility. Before we proceed to analyze the binary, we have to learn some basic concepts about the `IDA Pro` scripting facility. `IDA Pro` scripts syntax are similar to `C` programming language. The syntax as follows:

1. `IDA Pro` scripts only recognize one type of variable, i.e. `auto`. There are no other variable types such as `int`, `char`, etc. The declaration of variable in an `IDA Pro` script as follows:

```
auto variable_name;
```

2. Every statement in an `IDA Pro` script ends with a semicolon (`;`), just like in the `C` programming language.

3. Function can return a value or not, but there no return type declaration. The syntax as follows:

```
static function_name(parameter1, parameter2, parameter_n, ...)
```
