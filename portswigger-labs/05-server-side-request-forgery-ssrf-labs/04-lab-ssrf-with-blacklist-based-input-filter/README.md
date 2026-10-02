# Lab: SSRF with blacklist-based input filter

https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter

- الفكره من ال Lab ده ان في parameter مصاب ممكن يخليني اعرف اوصل ل internal server  بس  في firewall ان في blacklist بتحظر نوع معين من الwords

```bash
1. stockApi=http://127.1/Admin

2. stockApi=http://127.1/Admin/delete?username=carlos
```

نفس آلية "Check stock" اللي فاتت (`stockApi` parameter)، بس دلوقتي لو حطيت:

```
stockApi=http://127.0.0.1/admin
```

هتلاقي error بيقولك إن الـ URL ده "blocked" أو غير مسموح. يبقى فيه فلتر شغال بيدور على strings زي `127.0.0.1` أو `localhost` ويرفضها.

المشكلة إن blacklist زي دي بتفتكر إنها غطّت كل الطرق للإشارة لنفس الـ server، لكن في الحقيقة فيه طرق كتير بديلة تشاور على نفس الـ localhost:

#### إزاي بتتحل

جرّب الـ bypasses دي واحدة واحدة (أو بالـ Intruder):

1. **استخدام IP بصيغة مختلفة بدل `127.0.0.1`:**

```
stockApi=http://2130706433/admin        (decimal representation)
stockApi=http://017700000001/admin      (octal representation)
stockApi=http://127.1/admin             (shorthand)
```

1. **إضافة credentials وهمية قبل الـ hostname (URL parsing trick):**

```
stockApi=http://username@localhost/admin
```

بعض الـ parsers بتفتكر إن اللي بعد `@` هو الجزء المهم، لكن بعضها التاني بيفهمه غلط ويوصل فعليًا للـ localhost.

1. **استخدام domain بيعمل redirect لـ `127.0.0.1`** (أقوى وأشهر طريقة في الـ lab ده تحديدًا):
    - سجّل أو استخدم domain جاهز زي:

```
   http://localtest.me
```

ده بيعمل resolve تلقائيًا لـ `127.0.0.1`، فمن غير ما يكون فيه كلمة "localhost" أو "127.0.0.1" في الـ string، السيرفر بيوصل فعليًا للـ localhost.

بديل تاني (وهو المتبع في حل الـ lab الرسمي): تعمل domain بتاعك يعمل **HTTP redirect** لـ `http://127.0.0.1/admin` (مثلاً باستخدام موقع زي `burpcollaborator` مع redirect rule أو أي redirect service)، عشان الـ filter يشوف الـ domain الأولاني (اللي مش فيه كلمة blacklisted)، لكن بعد الـ redirect، السيرفر يبقى فعليًا بيوصل لـ `127.0.0.1/admin`. الـ filter بيفحص الـ URL الأصلي بس، مش بيتابع الـ redirect.

1. لما توصل لصفحة الـ admin، استخدم رابط الحذف زي الـ labs اللي فاتت:

```
stockApi=http://127.1/admin/delete?username=carlos
```

(أو من خلال الـ redirect لو الـ IP-format البسيط برضو اتبلوك)

#### فايدته إيه

- بيوضح إن **blacklist-based filtering** غالبًا فاشلة كطريقة دفاع، لأن فيه طرق لا نهائية تقريبًا لتمثيل نفس الـ resource (encoding مختلف، IP formats مختلفة، DNS tricks، redirects...).
- بيعلّمك أهم تكتيك في bypass الـ SSRF filters: **DNS rebinding / open redirect** — إنك تخلي الـ validation يحصل على domain "بريء"، لكن الـ request الفعلي (بعد الـ redirect) يروح لمكان تاني.
- بيرسّخ مبدأ أمني مهم: الدفاع الصح ضد SSRF مش إنك "تمنع القيم الخطيرة"، لكن إنك تعمل **whitelist صارم** لسيرفرات/دومينات محددة مسموح بيها بس (وده موضوع الـ lab اللي بعده).
