
``` 
      \_l\_ckout\_string\_key---

                  |

      |

      5A0533EA  /\*计算第一个seed\*/

                  |

      |

      5A053547  /\*计算第二个seed\*/

                  |

      |

      5A0539DB <-

              |

  |

  5A0539CD---  /\*license比较\*/
```

现在我们可以修改`lm_code.h`并且编译`lmcrypt`。使用`3D4DA1D6h`这样的字节模式，我们在`AddInlm.dll`中找类似的位置。其实我们只需要`LC_CHECKOUT`作为feature名。使用其他的模式能得到相应的位置（对于没有flexlm SDK的朋友来说，比较有用）：

``` 
 jutil.dll          AddInlm.dll

      Selicwiz.exe

 5A0533EC

  046CAEDC         00428C6C

 .

5A0539CD        046CB4BD

   0042924D
```

The origin of the above code is the object `lm_ckout.obj` winthin the `lmgr.lib` library.（大体意思是上面的代码从那里来的）

``` 
|

       |

               5A0533EA

/\*计算第一个seed\*/

           |

5A053547  /\*计算第二个seed\*/

               |

   5A0539DB <-

           |      |

            5A0539CD---  /\*license比较\*/

       |

              --
```

翻译得不好，大虾莫笑:P
