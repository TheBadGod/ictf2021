# Imaginary

## Task

```python
#!/usr/bin/env python3

import random
from solve import solve

banner = '''
Welcome to the Imaginary challenge! I'm gonna give you 300 imaginary/complex number problems, and your job is to solve them all. Good luck!

Sample input: (55+42i) + (12+5i) - (124+15i)
Sample output: -57+32i

Sample input: (23+32i) + (3+500i) - (11+44i)
Sample output: 15+488i

(NOTE: DO NOT USE eval() ON THE CHALLENGE OUTPUT. TREAT THIS IS UNTRUSTED INPUT. Every once in a while the challenge will attempt to forkbomb your system if you are using eval(), so watch out!)
'''

flag = open("flag.txt", "r").read()
ops = ['+', '-']

print(banner)

for i in range(300):
	o = random.randint(0,50)
	if o > 0:
		nums = []
		chosen_ops = []
		for n in range(random.randint(2, i+2)):
			nums.append([random.randint(0,50), random.randint(0,50)])
			chosen_ops.append(random.choice(ops))
		out = ""
		for op, num in zip(chosen_ops, nums):
			out += f"({num[0]}+{num[1]}i) {op} "
		out = out[:-3]
		print(out)
		ans = input("> ")
		if ans.strip() == solve(out).strip():
			print("Correct!")
		else:
			print("That's incorrect. :(")
			exit()
	else:
		n = random.choice("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")
		payload = f"__import__['os'].system('{n}(){{ {n}|{n} & }};{{{n}}}')"
		print(payload)
		input("> ")
		print("Correct!")

print("You did it! Here's your flag!")
print(flag)
```

## Solution

This was just a programming task, so I coded some small stuff in python

```python
from pwn import *

r = remote("chal.imaginaryctf.org", 42015)

def solve_nums(s):
    try:
        reals = 0
        imags = 0
        s = s.decode()
        s = s.replace(" ", "")
        y = [x.strip() for x in s.split(")") if x.strip() != ""] # each part
        nums = []
        for i in y:
            start = 0
            neg = False
            allneg = False
            if i[0] == '-':
                allneg = True
                start += 1
            elif i[0] == '+':
                allneg = False
                start += 1

            if i[start] == '-':
                neg = True
                start += 1
            elif i[start] == '+':
                neg = False
                start += 1
            else:
                start += 1

            cur = start
            while i[cur] != '+' and i[cur] != '-':
                cur += 1
            x0 = int(i[start:cur]) * (-1 if neg!=allneg else 1)
            print(x0)

            if i[cur] == '-':
                neg = True
                start = cur + 1
            elif i[cur] == '+':
                neg = False
                start = cur + 1

            x1 = int(i[start:-1]) * (-1 if neg!=allneg else 1)
            print(x1)

            reals += x0
            imags += x1

        op = "+" if imags > 0 else ""
        return (f"{reals}{op}{imags}i").encode()
    except Exception as e:
        print(e)
        return b""

r.recvuntil(b"watch out!)")

r.recvline()
r.recvline()
for i in range(300):
    x = solve_nums(r.recvline())
    r.sendline(x)
    r.recvline() # correct
r.interactive()
```


I know, way too convoluted, but I thought that maybe 
there will also be more complex (pun intended) calculations.

`ictf{n1c3_y0u_c4n_4dd_4nd_subtr4ct!_49fd21bc}`
