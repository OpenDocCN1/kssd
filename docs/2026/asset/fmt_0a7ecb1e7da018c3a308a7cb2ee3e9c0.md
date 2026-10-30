
```
                                     =-------------------------=

                                     | [0x4003b0]              |

                                     | main:                   | // Main Function

                                     | (fcn) sym.main 260      |

                                     | addiu sp, sp, -0x28     |

                                     | sw ra, 0x24(sp)         |

                                     | sw fp, 0x20(sp)         |

                                     | move fp, sp             |

                                     | lui gp, 0x42            |

                                     | addiu gp, gp, 0x680     |

                                     | sw gp, 0x10(sp)         |

                                     | sw a0, 0x28(fp)         | // Store argc

                                     | sw a1, 0x2c(fp)         | // Store argv[]

                                     | lw v0, 0x28(fp)         | // Load argc into the $v0 register

                                     | slti v0, v0, 2          | // Set $v0 to 1 if the value of $v0 is less than 2, otherwise set it to 0.

                                     | beqz v0, 0x400418 ;[a]  | // If $v0 is equal take the "true" branch. (Branch if Equal to Zero)

                                     | nop                     |

                                     =-------------------------=

                                           t f

                           .---------------' '--------------------------------.

                           |                                                  |

                           |                                                  |

                     =-------------------------=                  =-----------------------=

                     |  0x400418               |                  |  0x4003e4             |

                     | lw v0, 0x2c(fp)         | // load argv[]   | lw v0, 0x2c(fp)       |

                     | addiu v0, v0, 4         | // offset +4     | lw v1, (v0)           |

                     | lw v0, (v0)             | // load argv[1]  | lui v0, 0x40          |

                     | move a0, v0             |                  | addiu a0, v0, 0x70f0  | // Load Format String

                     | lw v0, -0x7fd0(gp)      | // $v0 = atoi    | move a1, v1           | // Load argv[0]

                     | move t9, v0             | // $t9 = $v0     | lw v0, -0x7fd4(gp)    | // Resolve printf pointer and load it into $v0

                     | jalr t9  ; atoi         | // call atoi     | move t9, v0           | // Move $v0 into $t9

                     | nop                     |                  | jalr t9   ; printf    | // Call printf which is in $t9 (Jump and Link Register)

                     | lw gp, 0x10(fp)         |                  | nop                   |

                     | sw v0, 0x18(fp)         | // int a         | lw gp, 0x10(fp)       |

                     | lw v0, 0x18(fp)         | // load int a    | addiu v0, zero, 1     | // Return 1

                     | slti v0, v0, 5          | // if < 5        | j 0x40049c ;[d]       | // Exit function

                     | beqz v0, 0x400478 ;[b]  | // true if $v0=0 | nop                   |

                     | nop                     |                  =-----------------------=

                     =-------------------------=                             v

                           t f                                               |

             .-------------' '---------------------------------.             '--.

             |                                                 |                |

             |                                                 |                |

  =-----------------------=                       =-----------------------=     |

  |  0x400478             |                       |  0x400450             |     |

  | lui v0, 0x40          |                       | lui v0, 0x40          |     |  // Block 0x400450 does the same as 0x400478

  | addiu a0, v0, 0x7118  | // load format string | addiu a0, v0, 0x7104  |     |  // Just needs to jump due to where it sits.

  | lw a1, 0x18(fp)       | // int a printf arg   | lw a1, 0x18(fp)       |     |

  | lw v0, -0x7fd4(gp)    | // load printf in $v0 | lw v0, -0x7fd4(gp)    |     |

  | move t9, v0           | // move $v0 to $t9    | move t9, v0           |     |

  | jalr t9  ; printf     | // call $t9 (printf)  | jalr t9 ; printf      |     |

  | nop                   |                       | nop                   |     |

  | lw gp, 0x10(fp)       |                       | lw gp, 0x10(fp)       |     |

  =-----------------------=                       | j 0x400498 ;[c]       |     | // Jump to Return 0 due to memory position

      v                                           | nop                   |     |

      |                                           =-----------------------=     |

      '-----------------.                             v                         |

                        .-----------------------------'                         |

                        |                                                       |

                        |                                                       |

                    =----------------=                                          |

                    |  0x400498      |                                          |

                    | move v0, zero  |  // Return 0                             |

                    =----------------=                                          |

                        v                                                       |

                        '-------------.-----------------------------------------'

                                      |

                                      |

                                  =---------------------=

                                  |  0x40049c           |

                                  | move sp, fp         |

                                  | lw ra, 0x24(sp)     |

                                  | lw fp, 0x20(sp)     |

                                  | addiu sp, sp, 0x28  |

                                  | jr ra               | // Function be gone!

                                  =---------------------=
```

每当我们做比较时，我们就可以看到 `slti` 指令的用法。 由于大量的比较运算符和类型，条件语句将成为学习新汇编语言中最耗时的部分。 请参考 C 参考文献，以确保您分析了生成条件分支的所有不同方法。 例如在 MIPS 中，有些条件语句使用可能被滥用的有符号或无符号立即数。

现在你已经看到了一些例子，只要你有一个编译器和反汇编器，你就应该有能力来学习任何处理器的架构和汇编。 如果你不这样做，那么你将不得不以更加艰难的方式去做，阅读处理器的开发者手册，最终设计你自己的汇编器，模拟器和反汇编器。

# GPL

如果您正在审计使用开源软件的设备，该软件可能会根据通用公共许可证进行许可。 如果是这样，那么简而言之，如果开发人员使用代码并编译它，则必须提供源代码！ 如果开发者拒绝开源，那么他们就违反了 GPL。

考虑到这一点，许多路由器或其他小型设备都使用 Linux（或 FreeRTOS），Busybox 和其他利用 GPL 的开源软件。 因此，在开始反汇编之前，尝试使用短语“$供应商源代码”或“$产品源代码”进行简单的 Google 搜索。 以下是我只是通过 Google 搜索发现的一些示例源代码库。

[Samsung Open Source Release Center](https://bbs.kanxue.com/elink@7f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0H3k6h3&6K6L8%4g2J5j5$3g2Q4x3X3g2K6j5h3#2K6N6h3&6Y4i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2U0k6i4m8@1K9h3!0F1i4K6u0W2k6r3)9`.)

[ASUS RT-AC68U Source Code](https://bbs.kanxue.com/elink@77bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2M7%4g2K6i4K6u0W2j5$3!0E0i4K6u0r3N6i4y4Q4x3V1k6z5k6i4c8%4L8%4u0C8K9h3&6Y4i4K6u0r3f1W2c8m8b7K6j5^5g2g2)9J5c8V1S2W2L8s2m8p5k6i4y4C8i4K6g2X3c8r3!0%4L8X3I4G2j5h3c8Q4x3V1j5`.)

[Belkin Open Source Code Center](https://bbs.kanxue.com/elink@e53K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0W2L8r3E0A6L8W2)9J5k6h3y4G2L8g2)9J5c8Y4g2K6i4K6u0r3M7%4g2H3M7r3!0J5N6q4)9J5k6r3q4J5N6r3W2U0L8r3g2Q4x3@1k6S2M7Y4c8A6j5$3I4W2e0Y4g2E0i4K6y4p5y4e0p5J5x3K6R3`.)

[GPL Code Center - TP-LINK](https://bbs.kanxue.com/elink@12eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8H3i4K6u0V1L8r3W2F1K9#2)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0r3k6%4m8D9i4K6u0V1j5$3!0V1k6g2)9J5k6h3S2@1L8h3H3`.)

现在您应该了解如何轻松利用 Google 或您最喜爱的搜索引擎来查找您正在审计设备的源代码。

# 利用
