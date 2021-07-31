# Password Checker

## Task

I seem to have forgotten my password. Before I lock myself out, I immediately went to my password checker that I coded, but it amounted to no avail. I guess I will leave it to the experts.

Wrap your flag in ictf before submitting.

[password checker](http://password-checker.chal.imaginaryctf.org)

## Solution

We press F12, go into the debugger and see a big blob of obfuscated javascript code.
Luckily we can just copy the part which would get evalled and run it to get the
actual code inside. After inserting newlines where necessary into that code and
some debug prints to know which element in an array is which, I decoded the last
function in that mess into this:

```javascript
function boop(kek) {
    var _0x842c = _0x5980c6,
        _0x9be0fd = _0x58abf4,
        _0x20d591 = _0x1833,
        _0x32db26 = _0x9be0fd(0x1a5)[_0x20d591(0xc1)](_0x4594[0x2c]),
        _0x41414f = 0x0;
    while (!![]) {
        switch (_0x32db26[_0x41414f++]) {
            case _0x4594[0x2d]:
                var weirdsum = 0x0;
                continue;
            case _0x4594[0x2e]:
                for (var i = 0x0; i < kek[_0x842c(0x1b9)]; i++) {
                    charsum += kek.charCodeAt(i),
                        xorsum ^= kek.charCodeAt(i),
                        weirdsum = (weirdsum << 0x5) - weirdsum + kek.charCodeAt(i)
                };
                continue;
            case _0x4594[0x2f]:
                var xorsum = 0x0;
                continue;
            case _0x4594[0x30]:
                var charsum = 0x0;
                continue;
            case _0x4594[0x31]:
                console[_0x20d591(0xcb)](charsum, xorsum, weirdsum);
                !/[^ZYPH3NAFUR1GT_BMKLE.0]/.test(kek) &&
                    charsum == 0x7e8 &&
                    xorsum == 0x7e &&
                    weirdsum == -0x28dd475e &&
                    kek.charCodeAt(0x0) - kek.charCodeAt(0x1) == 0x1 &&
                    Math.floor(kek.charCodeAt(0x1) * kek.charCodeAt(0x2) * kek.charCodeAt(0x3) / 0x736) == 0x115 &&
                    kek.charCodeAt(0x4) + kek.charCodeAt(0x5) - kek.charCodeAt(0x6) + kek.charCodeAt(0x7) == 0x72 &&
                    Math.floor(kek.charCodeAt(0x8) * kek.charCodeAt(0x9) / kek.charCodeAt(0xa) * kek.charCodeAt(0xb)) == 0xcb1 &&
                    Math[_0x9be0fd(0x19c)](kek.charCodeAt(0xd) / kek.charCodeAt(0xc) / kek.charCodeAt(0xe) * kek.charCodeAt(0xf)) == 0x1 &&
                    (kek.charCodeAt(0x10) &&
                        kek.charCodeAt(0x12) == _0x20d591(0xc6)[_0x9be0fd(0x1b0)] << 0x2) &&
                    kek.charCodeAt(0x6) == kek.charCodeAt(0x11) &&
                    Math.floor(kek.charCodeAt(0x13) * kek.charCodeAt(0x14) / kek.charCodeAt(0x15)) == 0x2e &&
                    kek.charCodeAt(0x16) + kek.charCodeAt(0x17) - kek.charCodeAt(0x18) == 0x74 &&
                    kek.charCodeAt(0x19) + kek.charCodeAt(0x1a) + kek.charCodeAt(0x1b) == 0x8a &&
                    console.log("Congrats!!!");
                continue
        };
        break
    }
}
```

This is the function which is called when we click on the check button.
So we see, that we just have a bunch of constraints. So it's time to ask
our friend z3.

Here's my cracking script:

```python
from z3 import *

inp = [BitVec(f"i_{i}", 8) for i in range(0x1c)]

# .013ABEFGHKLMNPRTUYZ_
charset = "ZYPH3NAFUR1GT_BMKLE.0"

s1 = 0
s2 = 0
s3 = 0
s = Solver()
for i in inp:
    s1 += ZeroExt(16, i)
    s2 ^= i
    s3 = ((s3 << 5)&0xFFFFFFFF) - s3 + ZeroExt(64, i)

    s.add(Or([i == ord(c) for c in charset]))

for i in range(len(inp)-1):
    s.add(Implies(inp[i] == ord('_'), inp[i+1] != ord('_')))

s.add(s1 == 0x7e8)
s.add(s2 == 0x7e)
#s.add(s3 == -0x28dd475e)
s.add(s3 & 0xFFFFFFFF == 0xd722b8a2)

for i,c in enumerate("ZYPH3N_P1Z_F0"):
    s.add(inp[i] == ord(c))

s.add(inp[0] - inp[1] == 1)
s.add(ZeroExt(32, inp[1]) * ZeroExt(32, inp[2]) * ZeroExt(32, inp[3]) / 0x736 == 0x115)
s.add(inp[4] + inp[5] - inp[6] + inp[7] == 0x72)
s.add(ZeroExt(32, inp[8]) * ZeroExt(32, inp[9]) * ZeroExt(32, inp[11]) / ZeroExt(32, inp[10]) == 0xcb1)
s.add(ZeroExt(32, inp[13]) * ZeroExt(32, inp[15]) / (ZeroExt(32, inp[12]) * ZeroExt(32, inp[14])) == 0x1)
s.add(inp[16] != 0)
s.add(inp[18] == 21*4)
#s.add(inp[19] == ord('H'))
#s.add(inp[20] == ord('0'))
s.add(inp[6] == inp[17])
s.add(ZeroExt(32, inp[19]) * ZeroExt(32, inp[20]) / ZeroExt(32, inp[21]) == 0x2e)
s.add(inp[22] + inp[23] - inp[24] == 0x74)
s.add(inp[25] + inp[26] + inp[27] == 0x8a)

while s.check() == sat:
    m = s.model()
    s.add(Or([i != m[i].as_long() for i in inp]))
    print("".join([chr(m[i].as_long()) for i in inp]))
else:
    print("nay")
```

So first I say that each character needs to be in the charset as
specified in the checker. (We check that no character is not in the
set, which means that all characters need to be in it). Then
I calculate the same sums and restrict them to be what it should be
Then all the other checks are also added. Make sure for multiplications
and sums where there could be an overflow to ZeroExtend the numbers
to have enough bits. Also make sure to rearrange the divisions in
a way such that it has the same effect as a Math.floor in javascript.

This script will then produce all solutions over time. Since it was
too slow I disabled the third sum check (The one I called weirdsum in
the javascript code) and ran it again, looked for words which would make
sense and restricted the first few characters to that word and continued
that until I got the actual solution by enabling the third check again

`ictf{ZYPH3N_P1Z_F13FRT_T0RTUR3...}`
