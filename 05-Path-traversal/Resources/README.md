# PATH TRAVERSAL

## What is a 'Path Traversal'?

This vulnerability allows reading arbitrary files on the server filesystem by manipulating a parameter that builds a file path, using `../` sequences — each one meaning "go up one folder level".
The idea is to exploit a file path that the application uses to display content, in order to escape the intended folder and access any file elsewhere on the server.

For example, if an application builds a path like:
```
/var/www/html/pages/
```
and we manipulate it by appending `../` sequences followed by a target file:
```
/var/www/html/pages/../../../../etc/passwd
```
We escape the `pages/` folder entirely and reach a completely different file located elsewhere on the disk. That is a security vulnerability.

### Local File Inclusion (LFI)

LFI is a direct consequence of Path Traversal. Instead of simply reading the content of a file, the application actually **includes and executes it as code** (via PHP functions like include() or require()).

Path Traversal alone -> Read the raw content of a file
Local File Inclusion (LFI) -> Read + execute the file as code


## How to find the vulnerability?

To find the flag of this vulnerability you need to try one of the basics test for hacking.
<pre>
/etc/passwd
</pre>

`/etc/passwd` is the standard proof-of-concept target for Path Traversal because:
- It exists by default on every Linux system
- It is always located at the same path
- It is readable by any user on the system (public read permissions) \

*More info about standard hacking test targets in the general README.*

On the main page, you need to test by adding the /etc/passwd with a suit of '../' sequences until you successfully go back up out of the folder.

<pre>
http://[IP]/index.php
</pre>
to
<pre>
http://[IP]/index.php?page=../../../../../../../../etc/passwd
</pre>

The principle: the server did not strictly validate the number of directory levels, nor did it properly neutralize the `../` sequences. This allowed us to escape the intended folder and access system files.

Before reaching the correct number of levels, you may encounter popups saying **"WTF?"** or **"Almost"**, these are good signs that you are on the right track!
They are triggered either because the developer added partial filtering that detects the attempt level until root folder, or because the requested file does not exist at that particular depth yet.


## Suggestions to protect from Path Traversal

### Why it was a vulnerability?

The `?page=` parameter was used directly to select which file to display, without any strict validation of its value. This meant that an attacker could freely navigate the server's filesystem.

Even though `/etc/passwd` no longer contains passwords (those moved to `/etc/shadow` decades ago), it still exposes:
- **All system usernames**, which can be used in brute-force attacks against SSH or other services
- **The shell assigned to each account**, which reveals which accounts belong to real human users vs. system accounts

This information should not fall into the wrong hands!


### How to do protect better?

**1. Whitelist**

Instead of trying to block dangerous patterns like `../` we define in advance the exact list of allowed page names and reject everything else.
Maintaining a **blacklist** (blocking what you know is dangerous) is always incomplete. A **whitelist** (only allowing what you know is safe) is far more robust. This is a general security principle that applies everywhere!

```php
$allowed_pages = ['survey', 'member', 'signin', 'recover', 'upload'];

if (in_array($_GET['page'], $allowed_pages)) {
    include($_GET['page'] . '.php');
} else {
    die("Page not found");
}
```

**2.`basename()` or `realpath()` to neutralize traversal attempts**

These PHP functions resolve or strip directory components from a path, neutralizing `../` sequences regardless of how they are written (encoded, doubled, mixed, etc.):

```php
$page = basename($_GET['page']);
include($page . '.php');
```

**3. Restrict permissions at the system level (defense in depth)**

Configure the web server so that the PHP process does not have read access to files outside its own folder. Even if a code vulnerability exists, the server will refuse to access files outside the defined perimeter.

```ini
; in php.ini
open_basedir = /var/www/html/
```


### Conclusion

**Never trust user input!**

The most reliable protection against Path Traversal is a strict whitelist of allowed values. Functions like **basename()** or **realpath()** combined with path verification add further layers of protection. Server-level restrictions such as PHP's **open_basedir** directive provide an additional defense even if the application code itself contains a flaw. \
This vulnerability falls under **OWASP A01:2021 – Broken Access Control**, which covers all cases where an application fails to properly restrict access to resources that should be protected.
