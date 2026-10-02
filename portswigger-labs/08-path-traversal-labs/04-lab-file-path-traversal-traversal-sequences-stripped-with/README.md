# Lab : File path traversal , traversal sequences stripped with superfluous URL-decode

https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode

- الفكره من الLab ده ان اموقع بيعتمد علي ازاله الsequence  الخاص ب path traversal فا انا ممكن استخدم ال decode علشان اتخطي الحمايه بس هو مش بيقبل بس decode واحد   دا عاوز ال double decode

```bash
1. Basic encoding: %2e%2e%2f for ../

2. Double encoding: %252e%252e%252f decodes to %2e%2e%2f

3. GET /image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%2fpasswd
```

زي كل معامل الـ path traversal، التطبيق بيجيب صورة منتج بناءً على قيمة `filename` جايه من المستخدم، وبيحاول يحمي نفسه بإنه يشيل أي `../` (traversal sequences) من القيمة دي قبل ما يستخدمها.

### المشكلة في اللاب ده تحديدًا (Superfluous URL-decode)

هنا السيرفر بيعمل حاجتين بترتيب غلط:

1. **الأول**: بيشيل أي `../` موجودة في الـ input (زي كل اللابات التانية).
2. **بعد كده**: بيعمل **URL-decode** تاني للـ input (عملية decode زيادة عن الحاجة — "superfluous" يعني "زايدة عن اللزوم").

المشكلة إن الفلترة (إزالة `../`) بتتنفذ **قبل** عملية الـ decode. يعني لو انت بعتّ الـ `../` مُشفّرة (URL-encoded)، الفلتر مش هيلاقيها كـ `../` أصلاً وقت الفحص، وهيسيبها زي ما هي. بعدين السيرفر بيعمل decode لها فتتحول لـ `../` فعلية **بعد** ما الحماية خلصت شغلها.

### الـ Payload المستخدم

بدل ما تبعت:

```
../../../etc/passwd
```

تبعت النسخة المشفرة (URL-encoded) بتاعتها:

```
..%2f..%2f..%2fetc/passwd
```

حيث:

- `%2f` = الترميز الخاص بـ `/` (Slash) في الـ URL encoding

### خطوات الحل عمليًا (Burp Suite)

1. **اعترض الريكوست** بتاع صورة المنتج باستخدام Burp Proxy:

```
   GET /image?filename=product1.jpg HTTP/1.1
```

1. ابعته على **Repeater**.
2. غيّر قيمة الـ `filename` لتبقى:

```
   filename=..%2f..%2f..%2fetc/passwd
```

1. **مهم جدًا**: متعملش الـ encode بتاع الـ payload يدوي مرتين أو تسيب Burp يعمل auto-encode زيادة، عشان أنت عايز الـ `%2f` يوصل للسيرفر زي ما هي (مش تتحول لـ `%252f`). لو محتاج تتأكد، افتح Inspector في Burp وشوف قيمة الـ parameter كام مرة اتعملها encode.
2. ابعت الريكوست.
3. هتلاقي إن السيرفر:
    - شاف الـ input `..%2f..%2f..%2fetc/passwd`
    - دوّر على `../` فيها → **ملقهاش** (لأنها مكتوبة كـ `%2f` مش `/` فعلي)
    - عدى الفحص من غير تعديل
    - عمل decode للقيمة كلها → تحولت لـ `../../../etc/passwd`
    - استخدمها في فتح الملف → قرا `/etc/passwd` بنجاح
4. الـ Response هيرجع محتوى الملف:

```
   root:x:0:0:root:/root:/bin/bash
   ...
```

### ليه ده بيحصل تقنيًا

الكود شبه كده:

python

```python
filename = request.get("filename")     # ../../../etc/passwd بس مشفرة كـ %2ffilename = filename.replace("../", "") # مفيش ../ ظاهرة، فمفيش تغييرfilename = url_decode(filename)        # هنا بس بتتحول ../../../etc/passwdpath = "/var/www/images/" + filenamereturn open(path).read()
```

الترتيب الغلط (فلترة قبل الـ decode) هو أساس الثغرة. المفروض إن أي فلترة/تحقق يحصل **بعد** ما تتعمل كل عمليات الـ decoding/normalization، مش قبلها، عشان تتأكد إنك بتفحص القيمة الحقيقية النهائية اللي هتتستخدم فعليًا.

### الدرس المستفاد

- ترتيب العمليات (decode ثم validate) مهم جدًا في الأمان؛ فحص قيمة "خام" قبل ما تتحول لشكلها النهائي مش كافي.
- أي encoding زيادة (URL encoding، double encoding، Unicode encoding) ممكن يستخدم لتفادي فلاتر بسيطة.
- الحل الصح: تعمل decode/normalize للمسار **مرة واحدة وبالكامل** الأول، وبعدين تتحقق من الناتج النهائي إنه فعلاً جوه الفولدر المسموح بيه (canonical path validation).
