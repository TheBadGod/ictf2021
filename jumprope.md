# Jumprope

## Task

CENSORED and CENSORED Sitting in a tree, H-A-C-K-I-N-G! First comes pwn, Then comes rev, Then comes a flag And a happy dev!

[jumprope](https://imaginaryctf.org/r/BAE3-jumprope)

## Solution

We open the binary and see that it will print "CORRECT!"
if we input the correct flag. The checkFlag just xors
some stack variables with values from an int array and some
pseudo-random numbers.

Since we start at an offset of 8 in the stack, it's pretty
clear that we skip the RBP value and just go to the return values,
so we need to rop a few functions to print "CORRECT!" and use
the return addresses as our xor key with the constant array and
the generated values. If we look at the next function we see that
it just goes through the bits and if the index of the bit is prime,
we xor the bit with a variable and after all eight bits we shift the
old value right one and append the new bit at the top.

So we can generate our key like this:

```python
x=2
def next(x):
    for i in range(8):
        y = x
        l18 = 0
        for j in "01010110"[::-1]:
            if j == "1":
                l18 = l18 ^ (y & 1)
            y >>= 1
        x = (x>>1)|(l18<<7)
    return x&0xFF

for i in range(88):
    y = next(x)
    print(hex(y)[2:].rjust(2,'0'),end="")
    x = y
```

Then we need to extract the int array:

```python
from pwn import *
e = ELF("jumprope")
x = [u64(e.read(e.sym.x + i*8, 8)) for i in range(88)]
print(*[hex(i)[2:].rjust(2,'0') for i in x],sep="")
```

Now we just need the correct return addresses.
For the c it's pretty easy, we just jump to
`0x401211`, then we need to load a value into
rdi, so we search for a gadget and find one at
`0x40148b`, then the constant `0x1337c0d3`,
the address to the "o" function, which is
`0x40122e`, then the address to "r" twice,
`0x40125b`, the address to "e", `0x401278`
then another pop rdi, the constant
`0xdeadface` and finally the address to
"t", being `0x401295`.

So we just xor the first key, the second key and

`11124000000000008b14400000000000d3c03713000000002e124000000000005b124000000000005b12400000000000781240000000000011124000000000008b14400000000000cefaadde000000009512400000000000`

(Because endianess everything needed to be inverted)

to get the flag

`ictf{n0t_last_night_but_the_night_bef0re_twenty_f0ur_hackers_came_a_kn0cking_at_my_d00r}`


