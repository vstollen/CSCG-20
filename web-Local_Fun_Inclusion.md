# Writeup
In this challenge, we were given a website, where images could be uploaded and shared, together with the information, that the website is written in PHP.

After a short exploration I saw, that all sites of the website were called, by loading the main page `index.php` and embedding another site, defined by the URL parameter `site`. This was a first attack surface since the embedded side was also interpreted and executed as PHP code.

During this exploration, I also found a file named `flag.php`, which probably would contain the flag, but was only rendered to a blank page. So I had to find a way to view it, without executing it as PHP code.

Eine Angriffsfläche hierbei bietet der Dateiupload. Im Netz findet man zahlreiche vorgefertigte PHP-Webshells. PHP Programme, die Befehle in der Shell ausführen und anzeigen können. Also versuchte ich eine solche hochzuladen. Dies ging aber nicht so leicht, da die Seite die hochgeladenen Dateien auf die Dateiendung überprüft, wobei `.php` selbstverständlich nicht als valides Bildformat erkannt wurde. Desweiteren brachte auch das Umbenennen der Datei nichts, da die Datei trotz einer Validen Bildendung wohl auch auf den Inhalt überprüft und abgelehnt wurde.

So my idea was to use the image upload to upload a PHP program, which would then help me to access the flag. So after a short web search, I found out about PHP-Webshells. These are PHP programs, which give you remote shell access. My trivial idea to just upload one didn't work and resulted in an error message, telling me, that `.php` isn't a valid image format. Also renaming the file to `shell.jpeg` didn't work, since the upload also seemed to validate the file's data.

Luckily I found another [PHP-Webshell (written by jgor) on GitHub](https://github.com/jgor/php-jpeg-shell), which starts with an image header to bypass weak image verification checks. Uploading this finally worked. To execute it as a PHP I just embedded it into `index.php` using the `site` parameter. Using the webshell I then was able to copy and rename `./flag.php` into `./uploads/flag.txt`, which I was then simply able to access with my browser, revealing the Flag: `CSCG{G3tting_RCE_0n_w3b_is_alw4ys_cool}`.

# Security Threads
Two issues were leading to the flag.

1. The image verification in the file upload wasn't very strong.
2. We could execute PHP code by embedding it into `index.php`, by using the `site` parameter.

Together we got access to a shell, giving attackers the ability to access huge parts of the server itself.

# Possible Fix
These security breaches could be prevented, by (for example):

- A stronger image verification algorithm.
- Forbidding the execution of files in the `/uploads` folder.
- Not using `include()` in combination with URL parameters.
- Whitelisting the pages, that are allowed to be included.