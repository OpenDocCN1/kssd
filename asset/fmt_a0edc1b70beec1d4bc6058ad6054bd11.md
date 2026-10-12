
```
> 00CA5A89   |>  C785 74FEFFFF 08000000  mov dword ptr ss:[ebp-18C],8

> 00CA5A93   |. 817D 18 37B3D866       cmp dword ptr ss:[ebp+18],66D8B337

> 00CA5A9A   |. 75 0F                  jnz short LabVIEW.00CA5AAB
```

在`CA5A9A`处下断点，运行到此处时，`d 0xf30d29`

可以看到：

```
> 00F30D25  80000000  ...€

> 00F30D29  3E6299A9  

> 00F30D2D  A2000002   .
```

因此正确的`SIGN`就是`80A999623E02`。

`4dd466d0e1e0cc123afbe504da2462ca`
