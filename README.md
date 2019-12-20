# Bypass CSP against MIME Confusion Attack

Major browsers have implemented Content Security Policy against MIME confusion attacks, reported by CVE X, Y and Z, which use polyglot image files (GIF, JPG ...) with embedded JavaScript code (as described here:).
But based on my studies I found a way to bypass CSP and perform Cross-Site Script (XSS) attacks by exploiting the MIME Type of the HTTP header.
