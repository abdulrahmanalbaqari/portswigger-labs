# Lab : Server -Side template injection with a custom exploit

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-a-custom-exploit

- الفكر من ال Lab ده ان الموقع function الavatar and author  display مصابين فا اانا بستغل ان function الauhtor_display وانفز اوامر جوا function ال setAvatar وارجع اشوفها تاني فب صفحه الavatar

```bash
1. user.setAvatar()

2. user.setAvatar('/etc/passwd','image/jpg')

3. GET /avatar?avatar=wiener => هنا بتتعرض نتايج الاوامر الي بحطها في setavatar

4. user.setAvatar('/home/carlos/User.php','image/jpg') =>Lab  اطلع منه حل ال php  ده هيظهر ملف علشان  

5. user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpg')

6. user.gdprDelete()
```

## شرح الـ Lab: SSTI مع Chaining لقراءة وحذف ملفات (PortSwigger Academy)

الفكرة العامة: عندك ثغرة **Server-Side Template Injection (SSTI)** في خاصية "Preferred name"، وهتستغلها مش بس عشان تنفذ كود، لكن عشان توصل لـ **object** حقيقي في السيرفر (`user`) وتستدعي منه methods خطيرة زي `setAvatar()` و `gdprDelete()`. الهدف النهائي: قراءة ملفات حساسة، وفي الآخر حذف مفتاح SSH بتاع Carlos.

### خطوة بخطوة والسبب وراء كل واحدة

**1. تسجيل الدخول وعمل كومنت على بلوج، وفتح Burp كـ proxy**

- بتعمل كومنت عشان يبقى عندك صفحة تقدر "تعيد تحميلها" كل مرة تغير فيها الـ preferred name، لأن الـ template بتاع الاسم بيتنفذ (يترندر) وقت ما الصفحة اللي فيها الكومنت بتتفتح، مش وقت الحفظ نفسه.
- Burp Repeater هيكون أداتك الأساسية عشان تبعت نفس الـ request بتغييرات بسيطة كل مرة.

**2. اكتشاف إن "My account" فيه SSTI**

- زي ما شفت في lab سابق، حقل الاسم المفضل بيتفسر كـ template (زي Freemarker مثلاً)، يعني أي syntax تبعت فيها بيتنفذ كتعبير برمجي مش كنص عادي.
- المهم هنا: اكتشفت إنك تقدر توصل لـ **object اسمه `user`**، ده معناه إنك مش محصور في "طباعة نص"، لكن تقدر تنادي methods على object حقيقي جوه تطبيق الـ PHP/Java.

## ليه محتاجين الجزء ده `'image/jpg'`؟

الـ method `setAvatar()` في كود PHP بتاخد **باراميترين إجباريين**، مش واحد بس:

php

```php
setAvatar($path, $mimeType)
```

### السبب البرمجي (Technical)

لما جربت أول مرة تنادي الـ method بباراميتر واحد بس:

```
user.setAvatar('/etc/passwd')
```

السيرفر رجّعلك **خطأ (error)** بيقول إن الـ method محتاجة argument تاني وهو الـ **MIME type**، لأن الكلاس مصمم إنه لازم يحفظ نوع الملف (image/jpeg, image/png, image/gif...) جنب المسار، عشان لما الـ avatar يتعرض على الصفحة، السيرفر يعرف يحط الـ HTTP header الصح، زي:

```
Content-Type: image/jpg
```

يعني المفروض الـ method دي مصممة أصلاً لحفظ **صورة حقيقية**، فبتخزن حاجتين مع بعض:

1. مكان الملف (path)
2. نوعه (mime type) — عشان لما يترفع للمتصفح يتعرض صح كصورة

### ليه ده مهم في الاستغلال (Exploitation)

- التطبيق **مبيتحققش فعليًا** إن المسار اللي انت داخل بيه فعلاً صورة، ولا إن الـ mime type اللي انت كاتبه بيطابق نوع الملف الحقيقي.
- يعني انت بتـ"كدّب" على السيرفر وبتقوله: "صدقني الملف ده `/etc/passwd` هو صورة من نوع `image/jpg`" — والسيرفر بيصدقك من غير أي فحص حقيقي (زي فحص الـ magic bytes أو امتداد الملف).
- طالما وفرت الباراميترين اللي الـ function طالبالهم (المسار + النوع)، الكود بينفذ عادي من غير ما "يهتم" إن المحتوى الحقيقي مش صورة أصلاً.

**3. فحص خاصية رفع الـ Avatar**

- لما رفعت صورة غلط، رسالة الخطأ سربت لك حاجتين مهمتين (information disclosure classic):
    - اسم الـ method: `user.setAvatar()`
    - المسار: `/home/carlos/User.php`
- ده بيديك فكرة إن `user` object فيه method اسمها `setAvatar` بتاخد باراميترز معينة، وإن في ملف PHP فيه تفاصيل الكلاس ده ممكن تفيدك بعدين.

**4. رفع صورة صحيحة وتحميل صفحة الكومنت**

- ده عشان تتأكد إن الـ avatar شغال بشكل طبيعي الأول، وتفهم الـ flow قبل ما تكسّره.

**5. استخدام حقل `blog-post-author-display` في الـ SSTI لاستدعاء `setAvatar()` يدويًا**

```
user.setAvatar('/etc/passwd')
```

- هنا انت مش بتغير اسمك، انت بتستخدم الـ injection عشان تنادي method حقيقية على object السيرفر وتفرض عليه إنه "يحفظ" مسار ملف تاني كـ avatar بتاعك.
- الخطأ اللي هيرجعلك ("لازم تدي MIME type كـ argument تاني") ده بيوضحلك الـ signature الحقيقية للـ method — يعني عندك feedback loop بيساعدك تكتشف الـ API من غير ما تشوف الكود.

**6. تعديل الاستدعاء بإضافة الـ MIME type**

```
user.setAvatar('/etc/passwd','image/jpg')
```

- دلوقتي الـ avatar بتاعك (`wiener`) بقى "بيشاور" على `/etc/passwd` بدل الصورة الحقيقية.

**7. تحميل `GET /avatar?avatar=wiener`**

- التطبيق بيقرأ الملف اللي الـ avatar object بيشاور عليه ويرجّعه كـ response — وبما إن مفيش أي تحقق إن المسار ده لازم يكون جوه فولدر الصور، بتقدر تقرأ أي ملف على السيرفر (Path Traversal / Arbitrary File Read عن طريق منطق التطبيق نفسه مش عن طريق الـ URL).

**8. تكرار نفس الخطوات لقراءة `/home/carlos/User.php`**

- المرة دي مش بتقرأ ملف نظام عام، لكن بتقرأ **كود مصدر التطبيق نفسه** بتاع كلاس الـ User، عشان تفهم باقي الـ methods المتاحة.
- في الملف ده اكتشفت وجود method اسمها `gdprDelete()` — واللي زي ما اسمها بيوضح، وظيفتها إنها تمسح الـ avatar بتاع اليوزر (جزء من "حق الحذف" بتاع الـ GDPR).

**9. استغلال `gdprDelete()` لحذف ملف حقيقي مش avatar**

- الخطوة الذكية هنا: بما إن `setAvatar()` مفيهاش أي تحقق على المسار، ومفيش تحقق إن الملف ده فعلاً صورة، تقدر "تخلي" الـ avatar بتاعك يشاور على أي ملف حساس تاني — مش عشان تقراه، لكن عشان بعد كده تنادي `gdprDelete()` اللي هتمسح "الـ avatar الحالي"، واللي هو فعليًا الملف اللي انت حددته:

```
user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpg')
```

- وبعدين نداء:

```
user.gdprDelete()
```

- ده بيخلي التطبيق يمسح مفتاح SSH الخاص بـ Carlos من على السيرفر، وده بيثبت إنك قدرت تستخدم منطق business logic بريء الظاهر (حذف صورة شخصية) عشان تعمل حذف ملف تعسفي (Arbitrary File Deletion).

### الدرس الأساسي من الـ Lab

- **SSTI** مش بس بيدي تنفيذ كود، ممكن كمان يدّيك وصول مباشر لـ objects حقيقية في السيرفر.
- رسائل الخطأ ممكن تسرب لك تفاصيل داخلية (اسم methods، مسارات ملفات) بتساعدك تبني الهجوم خطوة بخطوة.
- لما method زي `setAvatar()` بتاخد مسار من غير تحقق (validation)، بيتحول لأداة قراءة ملفات.
- ودمج ده مع method تانية زي `gdprDelete()` بيحولها لأداة **حذف** ملفات — وده مثال كويس على "chaining vulnerabilities": كل ثغرة لوحدها ممكن تبان بسيطة، لكن مع بعض بتدي تحكم كامل في نظام الملفات.
