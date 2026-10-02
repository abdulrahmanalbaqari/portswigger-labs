# Lab : File path traversal , validation of start of path

https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path

- الفكره من الLab ده ان الموقع بيبعت في  parameter الخاص بfile name  مش اسم الصوره بس لا دا المسار كله فا انا استغليته وخرجت منه واستدعيت الملفات الحساسه الي انا عاوزها

```bash
1. GET /image?filename=/var/www/images/../../../etc/passwd
```

في اللاب ده الفرق الأساسي عن باقي اللابات إن الـ `filename` parameter مش بيبعت اسم الصورة بس، لكن بيبعت **المسار الكامل (full path)** للملف. يعني الـ request بيكون شكله تقريبًا:

```
GET /image?filename=/var/www/images/product1.jpg
```

### المشكلة (الحماية الموجودة وليه مش كافية)

التطبيق بيتحقق من حاجة واحدة بس: إن الـ path اللي جاي من المستخدم **لازم يبدأ بـ** `/var/www/images/` (يعني الفولدر المسموح بيه). يعني الكود شبه:

python

```python
filename = request.get("filename")if filename.startswith("/var/www/images/"):    return open(filename).read()else:    return error()
```

المشكلة إن الفحص ده بيتأكد بس من **بداية** الـ string، ومبيعملش أي **normalization** أو resolve للمسار بعد كده. يعني طالما الـ string بيبدأ بـ `/var/www/images/`، مش مهم إيه اللي جاي بعد كده — حتى لو فيه `../` بتطلع منه تاني وتوصل لمكان تاني خالص.

### الـ Payload

```
/var/www/images/../../../etc/passwd
```

- الجزء الأول `/var/www/images/` بيخلي الفحص `startswith()` ينجح ✅
- بعد كده `../../../etc/passwd` بترجع لفوق 3 مرات من `/var/www/images/` وتوصل للروت، وتدخل على `/etc/passwd`

### خطوات الحل (Burp Suite)

1. **افتح اللاب** وتصفح صفحة منتج فيها صورة، وشغّل Burp Proxy وانت متصفح.
2. في **Proxy > HTTP history**، دوّر على الريكوست بتاع الصورة، هيكون شكله:

```
   GET /image?filename=/var/www/images/product1.jpg HTTP/1.1
```

ابعته لـ **Repeater**.

1. غيّر قيمة الـ `filename` لتبقى:

```
   filename=/var/www/images/../../../etc/passwd
```

يبقى الريكوست الكامل:

```
   GET /image?filename=/var/www/images/../../../etc/passwd HTTP/1.1
```

1. **ابعت الريكوست** (Send).
2. لاحظ الـ Response — هيكون فيه محتوى ملف `/etc/passwd`:

```
   root:x:0:0:root:/root:/bin/bash
   daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
   ...
```

بكده الـ lab يتحل ✅ (PortSwigger هيأكد الحل تلقائيًا لما يشوف محتوى الملف في الريسبونس).

### ليه الطريقة دي شغالة تقنيًا

نظام الملفات (filesystem) بيحلّ (resolve) أي `../` موجودة في المسار بغض النظر عن شكل الـ string الأصلي. يعني:

```
/var/www/images/../../../etc/passwd
```

لما يتحول لمسار فعلي (canonical path)، بيتحسب كده:

```
/var/www/images/    → ابدأ هنا
../                  → ارجع لـ /var/www/
../                  → ارجع لـ /var/
../                  → ارجع لـ /
etc/passwd           → /etc/passwd
```

فالنتيجة النهائية: `/etc/passwd`

الـ check بتاع `startswith("/var/www/images/")` بيبقى **صح تمامًا** على مستوى الـ string، لكن المسار الفعلي اللي هيتفتح مختلف كليًا، لأن السيرفر فحص الـ string **قبل** ما يعمل resolve للمسار.

### الدرس المستفاد

- التحقق من **بداية النص** (`startswith`) مش كافي أبدًا كحماية ضد path traversal، لأنه ممكن يتلاعب بيه بسهولة بإضافة أي حاجة بعده.
- الحل الصحيح: لازم تعمل **canonicalize** أو **resolve** للمسار الكامل الأول (يعني تحسب المسار الحقيقي النهائي بعد أي `../`)، وبعد كده تتحقق إن الناتج ده فعلاً بيبدأ بالفولدر المسموح بيه — مش تتحقق من الـ string الخام زي ما هو.
- الفحص لازم يكون على الـ **نتيجة النهائية**، مش على الـ **input الخام**.
