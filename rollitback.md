# Roll it back

## Task

```python
from itertools import *
from gmpy2 import *
def x(a,b):
    return bytes(islice((x^y for x,y in zip(cycle(a), cycle(b))), max(*map(len, [a, b]))))
def t(x):
    return sum((((x & 28) >> 4) & 1) << i for i, x in enumerate(x))
T = t(x(b"jctf{not_the_flag}", b"*-*")) | 1
with open("flag.txt", "rb") as f:
    flag = int.from_bytes(f.read(), "little")
    l = flag.bit_length()
print(f"{l = }")
for _ in range(421337):
    flag = (flag >> 1) | ((popcount(flag & T) & 1) << (l - 1))
print(f"{flag = }")

### Output
# l = 420
# flag = 2535320453775772016257932121117911974157173123778528757795027065121941155726429313911545470529920091870489045401698656195217643
###
```

## Solution

T is a constant, so all we really do is calculate a new bit based on the parity
of a few other bits. Which means we can restore the lowest bit of the previous state
by using the topmost bit of the current state and the other values which were used for
the calculation. Then we calculate the parity of that, invert it and append it as a new bit,
so we just need to modify the loop in the script we were given to get the flag:

```python
from itertools import *
from gmpy2 import *
def x(a,b):
    return bytes(islice((x^y for x,y in zip(cycle(a), cycle(b))), max(*map(len, [a, b]))))
def t(x):
    return sum((((x & 28) >> 4) & 1) << i for i, x in enumerate(x))
T = t(x(b"jctf{not_the_flag}", b"*-*")) # removed the | 1 here
flag = 2535320453775772016257932121117911974157173123778528757795027065121941155726429313911545470529920091870489045401698656195217643

# when going forwards we do new_bit=flag[-1] ^ flag[-6] ^ flag[-9] ^ flag[-10] ^ flag[-11] ^ flag[-14] ^ flag[-16] ^ flag[-18]
# we need to restore the flag[-1] bit, so we just xor the "new" bit (highest one) and then append that to the end...

for _ in range(421337):
    prev_popcnt = (flag >> 419)             # parity with lowest bit
    prev_state = (flag << 1) & (2**420-1)   # previous state without lowest bit
    num_ones = popcount(prev_state & T) & 1 # parity without lowest bit
    flag = prev_state | (num_ones ^ prev_popcnt)    # calculate lowest bit based on parities

flag = flag.to_bytes(53, "little")

print(f"{flag = }")
```

which gives us the flag

`ictf{I_did_not_have_linear_relations_with_that_LFSR}`
