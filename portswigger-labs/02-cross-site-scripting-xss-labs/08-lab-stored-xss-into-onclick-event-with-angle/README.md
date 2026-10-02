# Lab : Stored XSS into onclick event with angle brackets and double quotes HTML - encoded and single quotes and backslash escaped

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped

- الفكره من الLab ده انك تحفظ في قاعده البيانات رابط ملغم علشان اي حد يضغط عليه يجوله للصفحه الي انا عاوزها

```bash
1. http://foo?&apos;-alert(1)-&apos;
```
