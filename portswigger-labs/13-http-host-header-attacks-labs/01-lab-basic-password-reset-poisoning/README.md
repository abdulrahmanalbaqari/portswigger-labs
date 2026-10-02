# Lab : Basic password reset poisoning

https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning/lab-host-header-basic-password-reset-poisoning

- الفكره من الLab ده انك عاوز تغير الpassword الخاصه ب الtarget تمام فا انا محتاج الtoken  الخاص بالتغير الموجود في الheader و الtarget بيدوس علي اي ملف يعني الي النفروض اعمله اني اعمل redire للtarget علي صفحه تبعي بتقري الheader ذي  صفحه الexploite بتاعتي بتسمع في الlog page  المهم الخانه الي هفيرها هي الhost header

```bash
1. Host: exploit-0abf00ba046512328295e696017f00cc.exploit-server.net/
```
