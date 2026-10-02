# Lab : File path traversal , validation of extension with null byte bypass

https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass

- الفكره من الLab ده ان اموقع طالب تكون نهايه الامتداد هي نوع الملف ذي .jpg بس دي ممكن نتحايل عليها لو الموقع معول بلغه فديمه ذي C# c++ java  وكده عن طريق %00

```bash
1. GET /image?filename=../../../etc/passwd%00.jpg
```

زي اللابات السابقة، التطبيق بيجيب صورة منتج بناءً على `filename` جاي من المستخدم. هنا الحماية المستخدمة مختلفة عن الفلترة بتاعة `../` — بدل كده، السيرفر بيتحقق إن الـ filename **لازم ينتهي بامتداد صورة معروف** زي `.png`:

python

```python
filename = request.get("filename")if filename.endswith(".png"):    return open("/var/www/images/" + filename).read()else:    return error()
```

الفكرة إن المطور افتكر إن كده كفاية: "طالما الملف بينتهي بـ `.png`، يبقى هو صورة، مش ممكن يكون `/etc/passwd`".

### المشكلة (Null byte bypass)

المشكلة إن السيرفر ده مكتوب بلغة (زي Java القديم أو بعض إصدارات C/C++ اللي بتوصل بيها بعض الـ frameworks) بتتعامل مع الـ **null byte** (`\0` أو `%00` وقت الترميز في الـ URL) كنهاية للـ string.

يعني لو بعتّ:

```
../../../etc/passwd%00.png
```

- على مستوى **الـ validation code** (اللي بيفحص `.endswith(".png")`)، الـ string اللي بيتفحص هو النص الكامل، وهو بينتهي فعلاً بـ `.png` → الفحص بينجح ✅
- لكن لما القيمة دي توصل لدالة **فتح الملف** في نظام التشغيل (اللي بتتعامل مع null byte كـ terminator زي في لغة C)، الجزء اللي بعد `%00` (يعني `.png`) بيتجاهل تمامًا، ويتفتح الملف:

```
  ../../../etc/passwd
```

فبيحصل **تناقض بين الطبقتين**: طبقة الفحص شافت `.png` في الآخر فسمحت، لكن طبقة تنفيذ فتح الملف قطعت الـ string عند الـ null byte وفتحت ملف تاني خالص.

### خطوات الحل (Burp Suite)

1. **افتح Burp Suite** وشغّل الـ Proxy، وتصفح صفحة منتج فيها صورة في اللاب.
2. في **Proxy > HTTP history**، دور على الريكوست بتاع الصورة:

```
   GET /image?filename=product1.png HTTP/1.1
```

ابعته لـ **Repeater**.

1. غيّر قيمة الـ `filename` لتبقى:

```
   filename=../../../etc/passwd%00.png
```

يبقى الريكوست:

```
   GET /image?filename=../../../etc/passwd%00.png HTTP/1.1
```

1. **تنبيه مهم**: تأكد إن `%00` بتوصل زي ما هي (مش متشفرة زيادة لـ `%2500`). لو Burp بيعمل auto-encode، افتح الـ Inspector panel وشوف الـ raw value، أو استخدم Repeater direct edit في الـ request line.
2. **ابعت الريكوست**.
3. لاحظ الـ Response — لازم يكون فيه محتوى ملف `/etc/passwd`:

```
   root:x:0:0:root:/root:/bin/bash
   daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
   ...
```

بكده اللاب يتحل ✅.

### ليه ده بيحصل تقنيًا

- الـ validation layer (المكتوبة بلغة زي Java) بتشوف الـ string كسلسلة كاملة، وبتفحص آخر حروفها فعليًا → `.png` موجودة، فالفحص بينجح.
- لكن الطبقة الأقل مستوى (النظام الأساسي أو مكتبة native بتتفاعل مع الـ filesystem، زي بعض تطبيقات الـ file I/O المبنية على C) بتتعامل مع أي بايت null (`\0`) كـ **نهاية للـ string** (زي طبيعة الـ strings في لغة C أصلاً).
- بالتالي وقت فتح الملف، اللي بيتفتح فعليًا هو المسار **لحد الـ null byte بس**:

```
  ../../../etc/passwd
```

والباقي (`.png`) بيتجاهل تمامًا.

### الدرس المستفاد

- التحقق من **امتداد الملف** (`endswith(".png")`) مش حماية كافية ضد path traversal، لأنه بيتحقق من شكل النص بس مش من المسار الفعلي اللي هيتفتح.
- أي تناقض في طريقة تفسير الـ string بين طبقتين مختلفتين في السيستم (validation layer مقابل filesystem layer) بيفتح باب للـ bypass — ده مبدأ عام بيتكرر في ثغرات كتير مش بس path traversal.
- ملحوظة: الثغرة دي (null byte injection) قديمة نسبيًا وبقت نادرة في اللغات الحديثة (زي Java 7+ أو Python الحديث) لأنها بقت بتمنعها بشكل افتراضي، لكنها لسه موجودة في أنظمة legacy أو مكتبات native قديمة.
- الحل الصحيح: التحقق من الامتداد الحقيقي **بعد** ما تتأكد من الـ canonical path الكامل، مش من الـ string الخام، وكمان استخدام لغات/مكتبات حديثة بترفض الـ null bytes في أسماء الملفات من الأساس.
