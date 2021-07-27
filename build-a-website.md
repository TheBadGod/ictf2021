# Build-a-website

## Task

I made a website where y'all can create your own websites! Should be considerably secure even though I'm a bit rusty with Flask.

```python
#!/usr/bin/env python3

from flask import Flask, render_template_string, request, redirect, url_for
from base64 import b64encode, b64decode

app = Flask(__name__)

@app.route('/')
def index():
  # i dont remember how to return a string in flask so
  # here goes nothing :rooNervous:
  return render_template_string(open('templates/index.html').read())

@app.route('/backend')
def backend():
  website_b64 = b64encode(request.args['content'].encode())
  return redirect(url_for('site', content=website_b64))

@app.route('/site')
def site():
  content = b64decode(request.args['content']).decode()
  #prevent xss
  blacklist = ['script', 'iframe', 'cookie', 'document', "las", "bas", "bal", ":roocursion:"] # no roocursion allowed
  for word in blacklist:
    if word in content:
      # this should scare them away
      content = "*** stack smashing detected ***: python3 terminated"
  csp = '''<head>\n<meta http-equiv="Content-Security-Policy" content="default-src 'none'">\n</head>\n'''
  return render_template_string(csp + content)
```

## Solution

We notice that we do `render_template_string(csp + content)` so we can simply
do a jinja2 template injection, by making the content be `{{ something }}`.
We can either base64 encode the content ourself and make a direct connection
to /site or we can send the plaintext to /backend and that will handle the rest.

Our goal is to read the contents of the file called `flag.txt` on the server.
To do so, we can use anything except the word snippets in the blacklist.
So our first goal is get access to `__builtins__` to get to the open function.

We have multiple functions available, which have the `__globals__` object, so
we just take any function, in my example it was `url_for` and get the `__globals__`
attribute. But wait, "bal" is in the blacklist... How can we circumvent this?
Easy: We make use of the fact that in jinja we can access attributes via dict
notation `object["attrib"]` as well as the fact that in python two strings
directly after each other get concatenated. Thus we can do
`url_for["__globa""ls__"]` which gives us access to the globals, from there
it's smooth sailing. We can just take the `__builtins__` from the globals,
call `open` and then `read`

Our final payload looks like this:
`{{url_for["__globa""ls__"].__builtins__.open("flag.txt").read()}}`
