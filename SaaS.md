# SaaS

## Task

```python
from flask import Flask, render_template, request
import html
import os

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

blacklist = ["flag", "cat", "|", "&", ";", "`", "$"]

@app.route('/backend')
def backend():
    for word in blacklist:
        if word in request.args['query']:
            return "Stop hacking.\n"
    return html.escape(os.popen(f"sed {request.args['query']} stuff.txt").read())
```

## Solution

We see that we can't use many shell tricks, but we don't need to.
We can simply tell sed to output all the stuff by replacing
everything with itself. `'s/\(.*\)/\1/g'` will do that (by replacing
`.*` with a backreference to first thing that matched. Which must be the
whole text, because everything matches `.*`)

Now we just need to add a space and then specify the flag.txt file to read
from that. Sadly flag is blacklisted, but luckily `*` is not, which allows us
to do a shell expansion by doing `fla*`. So our payload looks like this now:

`'s/\(.*\)/\1/g' fla*`

and we get the flag:

`ictf{:roocu:roocu:roocu:roocu:roocu:roocursion:rsion:rsion:rsion:rsion:rsion:_473fc2d1}`
