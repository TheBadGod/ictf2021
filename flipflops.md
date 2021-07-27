# Flip Flops

## Task

```python
for _ in range(3):
	print("Send me a string that when decrypted contains 'gimmeflag'.")
	print("1. Encrypt")
	print("2. Check")
	choice = input("> ")
	if choice == "1":
		cipher = AES.new(key, AES.MODE_CBC, iv)
		pt = binascii.unhexlify(input("Enter your plaintext (in hex): "))
		if b"gimmeflag" in pt:
			print("I'm not making it *that* easy for you :kekw:")
		else:
			print(binascii.hexlify(cipher.encrypt(pad(pt, 16))).decode())
	else:
		cipher = AES.new(key, AES.MODE_CBC, iv)
		ct = binascii.unhexlify(input("Enter ciphertext (in hex): "))
		assert len(ct) % 16 == 0
		if b"gimmeflag" in cipher.decrypt(ct):
			print(flag)
		else:
			print("Bad")

print("Out of operations!")
```

`nc chal.imaginaryctf.org 42011`

## Solution

We see that we have AES in CBC mode, so we can just
encrypt something with at least two blocks (I just used 
31 null characters and one remaining for padding, 
which means a total of 62 zeros as plaintext). This gives
us some output which also has has 64 hex characters. We know
that when we decrypt this we get all nulls and then a one,
so we can xor the first block with the bytes of "gimmeflag".
This will give us a new input which when decrypted will result in
a weird first block (Will result in garbage most likely) but since
the decryption of the second block is only dependent on the previous
block's encrypted data, we don't need to worry about that.

So TLDR:
* Encrypt 31 null characters (== 62 zeros)
* Take output and xor the first few bytes with the bytes of "gimmeflag"
* Decrypt that to get the flag

`hex(encrypted^(int(b"gimmeflag".hex(),16)<<184))`

and when we send that, we get the flag (With one operation remaining):

`ictf{fl1p_fl0p_b1ts_fl1pped_b6731f96}`
