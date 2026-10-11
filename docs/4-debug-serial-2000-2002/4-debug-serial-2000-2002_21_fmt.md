
# CORE 的 official crackme

> 原文：[`www.pediy.com/kssd/pediy01/forum101.htm`](https://www.pediy.com/kssd/pediy01/forum101.htm)

`http://core.8bn.com/Download/cr-crack.zip`

这是 CORE 的 official crackme。记得一定要写注册机才算的，`patch` 不算！

## Reversing the CORE crackme

By Nolan Blender

Published by Tsehp Jan 9 2000

Since this program is a strainer, I'm not going to provide all the details on cracking this program. The general information that I will provide here may be helpful, however to build the required `keygen` you will still have to do the work yourself. This essay won't have too many specifics. I will discuss instead the strategies and thinking used to crack this program.

### Description of the program

The CORE crackme is a 77k program written by Egis from the CORE cracking group. This program is a small crackme which one must `keygen` in order to gain a trial membership in that group.

The program is designed in a way that makes it infeasable to extract a key by repetitive trial, or "brute forcing". The program is packed as well, so the program must be unpacked. Until recently, `procdump` was unable to automatically unpack the program, however recent versions of `procdump` can handle this.

`Icedump` was used to dump the process at a point where the program had been unpacked, but before the import table had been processed. The address of the Import Address Table was restored, and the location of the import table fixed.

At that point, `IDA` could be used on the program, and a map file was created. `MSYM` and `nmsym` were used to build an NMS file for `softice`, and full debugging could begin.

After some careful analysis, it was determined that there were two algorithms at work. The first algorithm took the user name and passed it through a non reversable algorithm which appears to be a variation on MD5.

The second algorithm took the entered key, consisting of four 32 bit values, and produced a second set of four 32 bit values. The values were then compared against the output of the MD5 hash, and if they were the same, the test passed, otherwise it failed.

Since MD5 is a non reversible algorithm, it's not possible to guess a name for a particular key. Besides, a `keygen` should be able to take arbitrary input to generate a key. It follows that the key processing algorithm must be reversible, or a brute force approach is available.

Careful examination of the algorithm makes it clear that the algorithm can't be avoided by brute forcing the solution - the algorithm works on 64 bit blocks and does many iterations. The first two values are first processed, then 1 and 2, then 2 and 3. A brute force attack is not practical on a 64 bit key. What's more, each pass overlaps the results of the previous pass, so it's not possible to partition the problem.

Here's what the encryption main loop does.

```
for (i = 0; i < 3; i++)
{
    val1 = 0x0badc0de;
    val2 = val1 / (0x50 + i);
    eg_encrypt(val2, &(arr1[i]));
}
```

A C emulation of this code section was written. It is easier to understand what is going on in C, so a reversing routine can be more easily written. Since the algorithm loops many times, the first part was to emulate what was happening for a single pass.

Reversing a single pass of the algorithm was the most time consuming part of this exercise, since it required making inferences about earlier values in the loop from later ones. There is a bit of trickiness going on in the encrypting routine, but the algorithm is fully reversible.

Once a single pass of the algorithm was completed, an algorithm which was the inverse of `eg_encrypt` was written. The decryption was run against the 128 bit output of the modfied MD5 algorithm, and a key was generated.

The hashing algorithm was ripped from the crackme, using `IDA`, modified to compile under `nasm`, and then linked against the code containing the decryption algorithm.

Many tools were used to crack this program. `SoftICE` 4.01 for debugging, `IDA` 4.01 for disassembly/map generation, `nasm` for code compilation, `UltraEdit32` 6.20 for code and object editing, `procdump` for editing the PE headers, and `icedump` to dump sections to disk. This is a good crackme to work on since many important cracker skills are exercised by this program. The algorithm is reversible, you must examine the program very carefully to see how it works at times.

Forward engineering tools were `nasm` to build the assembly code, and Microsoft Visual C++ 6.0 to build and link the `keygen`.

Special thanks to Miz, The Owl and VoxQuietis for their essays and assistance.

### Sample output

To join CORE, you have to supply the `keygen`.

```
D:\ckeygen>ckeygen

Nolan Blender's CoRE crackme keygen 1.0 (12-dec-1999)

Enter your name:

Nolan Blender

Generating your key...

Your key is: 7CB89309 32029722 BFFCF6C4 AF7D6DE0
```

---

这篇文章是高手 Nolan Blender 写的，是个老外。另外，我觉得看英文很 easy 的，可能是我看英文破解文章太多的缘故 (`dr0`)  `4dd466d0e1e0cc123afbe504da2462ca`
