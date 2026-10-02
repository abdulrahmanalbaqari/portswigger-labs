# Lab : File path traversal , traversal sequences blocked with absolute path bypass

https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass

- الفكره من الLab ده استغلالparameter ال filename علشان اعرف احقن payload الetc/passwd بس في intercept live step by step

```bash
1. GET /image?filename=/etc/passwd
```

### الفكرة العامة (Path Traversal)

التطبيق بيستخدم اسم ملف جاي من المستخدم (زي اسم صورة منتج) عشان يجيب الملف ده من على السيرفر، من غير ما يتحقق كويس إنه فعلاً جوه الفولدر المسموح بيه بس. ده بيسمح للمهاجم إنه "يهرب" من الفولدر ده ويقرا ملفات تانية حساسة في السيرفر، زي `/etc/passwd`.

### الفكرة الخاصة بالـ Lab ده

هنا السيرفر بيعمل حماية بسيطة: بيشيل أي `../` (traversal sequences) من الـ filename قبل ما يستخدمه، عشان يمنع حاجة زي:

```
filename=../../../etc/passwd
```

لكن الحماية دي **بتستهدف الـ relative paths بس**. لو بعت **absolute path** (مسار كامل من الروت) من غير أي `../` أصلاً، الفلترة معندهاش حاجة تشيلها، والسيرفر بيمرره زي ما هو ويفتح الملف مباشرة من الروت.

### خطوات الحل (زي ما انت كتبتها بالظبط)

1. **افتح Burp Suite** وشغّل الـ Proxy، وتصفح المنتجات في الموقع (الـ lab) وافتح صفحة منتج فيها صورة.
2. في تبويب **Proxy > HTTP history**، دور على الـ request اللي بيجيب صورة المنتج، هيكون شكله تقريبًا:

```
   GET /image?filename=product1.jpg HTTP/1.1
```

ابعته على **Repeater** (Ctrl+R أو Send to Repeater).

1. في الـ Repeater، **عدّل قيمة الـ `filename` parameter** خليها:

```
   filename=/etc/passwd
```

يعني الـ request يبقى:

```
   GET /image?filename=/etc/passwd HTTP/1.1
```

1. **ابعت الـ request** (Send) ولاحظ الـ Response.
2. لو الحل صح، الـ Response body هيكون فيه محتوى ملف `/etc/passwd` بدل الصورة، ومحتواه هيكون شكله كده:

```
   root:x:0:0:root:/root:/bin/bash
   daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
   bin:x:2:2:bin:/bin:/usr/sbin/nologin
   ...
```

بكده الـ Lab يبقى **Solved** ✅ (PortSwigger بيتأكد تلقائيًا إن محتوى الملف ظهر في الريسبونس).

### ليه الطريقة دي نجحت؟ (الشرح التقني)

الكود على السيرفر شبه:

python

```python
filename = request.get("filename")filename = filename.replace("../", "")   # الحماية الوحيدة الموجودةpath = "/var/www/images/" + filenamereturn open(path).read()
```

- لو بعتّ `../../../etc/passwd` → الفلتر بيشيل كل `../` وتبقى النتيجة `etc/passwd` بس، والمسار النهائي بيبقى `/var/www/images/etc/passwd` وده مش موجود → فشل.
- لو بعتّ `/etc/passwd` مباشرة → مفيش `../` أصلاً عشان الفلتر يشتغل عليها، فالـ filename بتفضل زي ما هي `/etc/passwd`. وبما إنها absolute path، أغلب دوال فتح الملفات (زي في Java، Python، PHP...) بتتعامل معاها كمسار كامل من الروت، فبتتجاهل أي حاجة اتحطت قبلها (`/var/www/images/`)، والنتيجة إن السيرفر بيفتح `/etc/passwd` الحقيقي.

### الدرس المستفاد

- إزالة `../` بس مش كافية كحماية، لأن فيه طرق تانية للتخطي (زي absolute paths، أو encoding، أو nested traversal `....//`).
- الحماية الصح: التأكد إن المسار النهائي بعد الـ resolve فعلاً **جوه** الفولدر المسموح بيه (canonical path check)، مش مجرد string replace لأنماط معينة.
