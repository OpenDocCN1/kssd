
# CloneCD

> 原文：[`www.pediy.com/kssd/pediy02/forum128.htm`](https://www.pediy.com/kssd/pediy02/forum128.htm)

Here is a Ivanopulo's .nfo part about asprotect:

"For those who's interested in this program's protection mechanism -

here's the info:

Main keycheck routine is implemented as Nyberg-Rueppel signature

scheme for Elliptic Curves defined over `F2^m` polynomial field. This

makes writing a keymaker for CloneCD impossible. (so don't believe if

someone says he managed to keygen it)

The curve used has 242 bit prime order

`1026DD85081B82314691CED9BBEC30547840E4BF72D8B5E0D258442BBCD31`

I have the data for field polynomial written down somewhere on my HD,

but atm I have no time to search for it, so sorry (the only thing I

can recall it was polynomial of 272 or 271 degree)

By the way, the curve used in CloneCD is the SAME curve defined over

the SAME polynomial field as used by WinRAR. Heh, I wonder where did

those guys find it and the library they used for implementing ECC. I

know only one library that implements ECC - Miracl, but it has only

`Fp` implementation. Maybe they bought the library from certicom...

anyway, if you have that library and don't mind to send it to me -

please do it. I had to dig into all this ECC implementation looking

at disassembled code. Not the most pleasant occupation I had in life

:)

And the second level of protection in this program is implemented

using ASProtect. Alex Solodovnikov did very good job on this

protector (hi Alex) but yet it is possible to defeat it. The point is

that ASProtect encrypts some program routines (in case of CloneCD

registration info/program settings saving functions) and decrypts

them with the password provided in the user's registration info

(encrypted there). But fortunatelly I was able to grab one of the

'stolen' keys and after it was decrypted in memory I knew ASProtect

password. Hehe, it's 'Suck me!' by the way.

Okay, enuff talking... use it and enjoy =)" `4dd466d0e1e0cc123afbe504da2462ca`
