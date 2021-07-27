# It's Not Pwn, I Swear!

## Task

I've made a challenge that's not pwn! Just because it looks like a bird and sings like a bird doesn't mean it's pwn. And good thing it's not pwn, because this binary has full protections (including a canary!).

[notpwn](https://imaginaryctf.org/r/0031-notpwn)

## Solution

We open the binary and see a typical bufferoverflow scenario, so
we use pwntools to....

overflow the buffer and overwrite the canary. So we go into the
stack check fail method which as we see is not the standard one
from libc, but an overwritten one, which later calls the one in libc.

So now we have a classical password check and it uses only two longs
which we overwrote, so we only need to send 10 bytes to get to the end
of the buffer and then something to make the checker happy.
I was way too lazy to calculate anything by hand, so I gave it to z3:

```python
from z3 import *

x = BitVec("i_10_18", 64)
y = BitVec("i_18_26", 64)

x0 = 0x6231726435333364

s = Solver()
a = x0 * x + y
s.add(a == 0x317ee37c444051c9)
a = a * x + y
s.add(a == 0x65bbfa1e87aa1f8d)

if s.check() == sat:
    print(s.model())
```

which gives us the two longs required. Then 
we convert them to ascii characters and voilà,
we get the password `th3c@n@ryh@sd!3d`, which
we input after 10 random characters to get the flag

`ictf{m@ry_h@d_@_pr3++y_b!rd_f3ath3rs_bri6ht_and_ye11ow_}`
