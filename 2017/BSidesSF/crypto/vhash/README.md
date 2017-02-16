# vhash

## Challenge

> ---- Due to a bug, the challenge might be easier than intended. Enjoy the free points! ----

> Can you gain admin access to this site?

> (The vhash binary is what's used for signing the cookie)

> http://vhash-c6bb0e85.ctf.bsidessf.net:9292

> [vhash.zip](vhash.zip)

## Solution

As the challenge notes, there is a bug in this challenge that makes it a very straight-forward solve: no crypto needed! `vhash.zip` contains 2 files:

- `index.php`- the login and cookie generation logic
- `vhash`- ELF executable to sign the cookie

After a walkthrough of the web application, it became clear the getting the flag required logging in as `administrator` by manipulating the cookie.

![login form](https://github.com/R3dCr3sc3nt/BSidesSF-2017/blob/master/crypto/vhash/login_form.png)
![login success](https://github.com/R3dCr3sc3nt/BSidesSF-2017/blob/master/crypto/vhash/login_success.png)

Inspecting the cookie manually confirms that the cookie information displayed in the webpage is correct.

Here is the relevant part of `index.php` used for cookie-setting.

```
  function do_hash($data) {
    $filename = tempnam(sys_get_temp_dir(), 'vhash');
    file_put_contents($filename, $data);

    $hash = substr(`/home/ctf/vhash $filename`, 0, 256);
    unlink($filename);

    return $hash;
  }

  function create_hmac($data) {
    return do_hash(SECRET . $data);
  }

  if(isset($_GET['action']) && $_GET['action'] === 'logout') {
    setcookie('auth', '');
    header('Location: index.php');
  }

  if(isset($_POST['username'])) {
    # Do pagey stuff
    if(is_valid($_POST['username'], $_POST['password'])) {
      # Create the cookie
      $cookie = 'username=' . $_POST['username'] . '&';
      $cookie .= 'date=' . date(DATE_ISO8601) . '&';
      $cookie .= 'secret_length=' . strlen(SECRET) . '&';

      # Sign the cookie
      $cookie = create_hmac($cookie) . '|' . $cookie;
      setcookie('auth', $cookie);
  }
```

The secret to solving the challege is finding the bug. After playing around with the vhash binary, I realized that vhash doesn't take a filename as an arguement, it takes a string. This means the the line ``$hash = substr(`/home/ctf/vhash $filename`, 0, 256);`` will always produce the same hash, no matter what the the `$data` in the file `$filename` is. Once I made this realization, it was just a matter for changing the `username` in the cookie to `administrator`.


This is the old cookie (from logging in as guest).

```
auth=ddd52d5a1d743847697929334ff2afc4a9cfbb21ebe5e6cd42b43f3e4cc9c625febc38a0dcc537740bf026a50fe16dc2e27a783fce6f3fbaf191df3080d5ab69457aaa31a331d5e0bfdc61d001597e473636c5077dacd8ee5563c93d46ccc00855c55461228376c8496f9013e316c80626e2499c7911d9a941dc0aa08ae63284|username=guest&date=2017-02-11T22:05:58+0000&secret_length=8&
```

This is the new cookie (set through the javascript console).

```
auth=ddd52d5a1d743847697929334ff2afc4a9cfbb21ebe5e6cd42b43f3e4cc9c625febc38a0dcc537740bf026a50fe16dc2e27a783fce6f3fbaf191df3080d5ab69457aaa31a331d5e0bfdc61d001597e473636c5077dacd8ee5563c93d46ccc00855c55461228376c8496f9013e316c80626e2499c7911d9a941dc0aa08ae63284|username=administrator&date=2017-02-11T22:05:58+0000&secret_length=8&
```

Refresh the page and the flag is revealed to be `FLAG:180e2300112ef5a4f23c93cfdec8d780`.
