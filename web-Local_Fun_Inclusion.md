# Writeup
Bei dieser Challenge war eine Website gegeben, auf die man Bilder hochladen und teilen kann, zusammen mit der Information, dass sie in PHP geschrieben wurde.

Nach kurzem erkunden viel auf, dass immer `index.php` zusammen mit einem URL-Parameter `site` aufgerufen wurde. Die übergebene `site` wird dann eingebettet in die Hauptseite angezeigt. Also kam mir die Idee, fremde Dateien durch das manuelle ändern des `site` Parameters anzuzeigen. Hierbei habe ich durch Zufall eine Datei `flag.php` gefunden, die aber eingebettet zu keiner Anzeige geführt hat. Ich musste also höchstwahrscheinlich den Dateiinhalt der `flag.php` ungerendert auslesen.

Eine Angriffsfläche hierbei bietet der Dateiupload. Im Netz findet man zahlreiche vorgefertigte PHP-Webshells. PHP Programme, die Befehle in der Shell ausführen und anzeigen können. Also versuchte ich eine solche hochzuladen. Dies ging aber nicht so leicht, da die Seite die hochgeladenen Dateien auf die Dateiendung überprüft, wobei `.php` selbstverständlich nicht als valides Bildformat erkannt wurde. Desweiteren brachte auch das Umbenennen der Datei nichts, da die Datei trotz einer Validen Bildendung wohl auch auf den Inhalt überprüft und abgelehnt wurde.

NOTES:

```
<?php

$FLAG = "CSCG{G3tting_RCE_0n_w3b_is_alw4ys_cool}";
```
`http://lfi.hax1.allesctf.net:8081/uploads/flag.txt`
 `https://github.com/jgor/php-jpeg-shell`
