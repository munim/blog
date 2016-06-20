---
layout: post
title: "Setup XDebug on XAMPP and Eclipse"
excerpt_separator: <!--more-->
tags:
    - eclipse
    - debug
---

Before proceeding with the setup, it’s recommended to note down few things which may save couple of hours.

- Your PHP version. Create a PHP file with `<?php phpinfo(); ?>` to get the version
- Your Windows version (32-bit or 64-bit). Check System Properties.

Now go to XDebug Download page and you will see Windows binaries with having some version information.

XDebug has different build for different version for PHP and OS, and the DLL file name is formatted like:

    php_xdebug-{xdebug-version}–{php-version}–{build-env}–{thread-safe/non-thread-safe}.dll


- xdebug-version: Generally, you should use the latest build provided by XDebug. Otherwise, use your required version.
- php-version: This section has the PHP version like 5.2 or 5.3 etc
- build-env: Generally you should consider using VC6 compiled version of XDebug but if you are using VC9 compiled PHP then you should consider using VC9 compiled version of XDebug.
- thread-safe/non-thread-safe: By default thread-safe builds does not have this marker where “nts” added for non-thread-safe. Use this if you have Zend Optimizer disabled. Generally you should use thread-safe.

Download the XDebug DLL and put it on your `{xampp-folder}\phpext` folder and rename the file to `php_xdebug.dll`. [NOTE: {xampp-folder} is just an alias. Consider it as where you installed XAMPP. Something like `C:\XAMPP`, if you’re using Windows]

Now you have to open `php.ini` located in your `{xampp-folder}\php` folder. Now search for `[XDebug]` section, if you find anything then just put the lines, otherwise add `[XDebug]` at the end of the file and the lines below:

    xdebug.remote_enable=1
    xdebug.remote_host="localhost"
    xdebug.remote_port=9000
    xdebug.remote_handler="dbgp"

Hold on, you are not done yet. One more line to make PHP know XDebug DLL.

If you are using less than PHP v5.3 then add

    zend_extension_ts={xampp-folder}phpextphp_xdebug.dll
	
If you are using PHP v5.3 and above then add

    zend_extension={xampp-folder}phpextphp_xdebug.dll

If you are using any custom build of PHP with `—enable-debug` then add
	
 zend_extension_debug={xampp-folder}phpextphp_xdebug.dll

OK! You are almost done. Search for `[Zend]` section now in `PHP.ini`, if you find `zend_extension_ts`, then comment the line using `;`. And that’s it!

Refresh you phpinfo page and search for `XDebug` and you will see setting of XDebug and you have successfully configured XDebug with PHP.

Considering you are using Eclipse v3.3 or above, which has built-in XDebug support. All you have to do is, go to “Debug Configuration” as set start URL, click “Apply” and close the window. Now click “Debug” button from toolbar to start debugging.

Points to note:

- There’s a XDebug bug which makes Apache server to crash while debugging with Eclipse. The crash occurs when invalid expressions (watch items) are there. Delete all expression items when terminating or launching your debug session.
- Always start your debug session with a PHP file, otherwise you may end up hanging on the message “Waiting for XDebug session”.

HAPPY LIFE :)