# Lines

## Task

```python
from Crypto.Util.number import bytes_to_long
import random

flag = bytes_to_long(open("flag.txt", "rb").read())
msg = bytes_to_long(b":roocursion:")

p = 82820875767540480278499859101602250644399117699549694231796720388646919033627
g = 2
a = random.randint(0, p)
b = random.randint(0, p)
s = pow(pow(g, a, p), b, p)

def encrypt(msg):
	return (s*msg) % p

print(f"{p = }")
print(f"{encrypt(flag) = }")
print(f"{encrypt(msg) = }")
```

p = 82820875767540480278499859101602250644399117699549694231796720388646919033627
e\_flag = 26128737736971786465707543446495988011066430691718096828312365072463804029545
e\_msg = 15673067813634207159976639166112349879086089811595176161282638541391245739514

## Solution

We can get s because we have `s*msg === e_msg   mod p`,
which means we can simply calculate the modular inverse
of the message modulo p and multiply the encrypted message
with that, calculate modulo p and get the value of s.

`s = (pow(msg, -1, p) * e_msg)%p`

this gives us a value of
47724061801079554597466585626373440191004658414760256322329566876897497386054
for s. Now we just calculate the modular inverse of that, 
then multiply the encrypted flag by that to get the
original flag.

`flag = (e_flag * pow(s, -1, p)) % p`

Now we just convert that to bytes and get

`ictf{m0d_4r1th_ftw_1c963241}`
