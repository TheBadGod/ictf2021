# Off to the Races!

## Task

They say that gambling leads to regrets, but we'll see. This online portal lets you bet on horse races, and if you can guess the admin password, you can collect all the money people lose, too. Maybe you'll collect enough to buy the flag?

* [races.py](https://imaginaryctf.org/r/D1DB-races.py)
* nc chal.imaginaryctf.org 42016

## Solution

The first thing I notices was the little regex puzzle,
so I went to regexr and tried to get a valid password
for the admin and get `ju5tneveeverl05e` but with the option
to put in more `eve` in the middle like this `ju5tneveeveeverl05e`.
So now we can log in as admin and make a horse win. 

We first have to win $100, we can do that by betting on three
horses, each with a bet of $100. Then we log in as admin and
declare a winner. This subtracts $100 once (the horse that won)
but adds $100 twice, netting in a total of positive 100.

Next we somehow need to be a normal user but still have access
to the admin menu. Luckily there's a toc-tou with the admin.value
variable. So if we can make the validation take more than two
seconds (Such that the login() method returns but we haven't reset the
admin flag) then we go into the admin menu, later when the verification
failed we get that message and are no longer admin, enabling us to get the
flag

Inputs:
```
1
1
100
1
2
100
1
3
100
2
ju5tneveeveeverl05e
1
4
ju5tneveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveeveverl05e
ictf{regrets_may_be_the_plural_of_regex_but_ive_no_regrets_from_betting_on_a_sure_thing}
```

and we get the flag:

You may need more or less "eve" in the password to logout, depending on how
fast the system is.

`ictf{regrets_may_be_the_plural_of_regex_but_ive_no_regrets_from_betting_on_a_sure_thing}`
