# Stings

## Task

Enter the beehive. Don't get stung.

(Note: the password/flag is in the format ictf{.\*})

## Solution

We open the file in ghidra, go to the main function.
We see that there's a stack variable, which get's filled
with printable characters, so we retype it to a char
array. Then we see that our input is just compared to
that array, but subtracting one from the constant first.

So we just take the constants, make sure the endianess
is correct, then subtract one from each byte and get the flag:

`ictf{str1ngs_4r3nt_h1dd3n_17b21a69}`
