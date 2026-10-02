# Lab : SSRF with whitelist-based input filter

https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter

- الفكره من الLab ده

### شرح الخطوات اللي بعتها (الحل الرسمي لـ Lab: SSRF with whitelist-based input filter)

الطريقة اللي بعتها دي هي الحل **الرسمي والأدق** بتاع الـ lab ده (بتفرق شوية عن الـ `@` trick اللي شرحته قبل كده، وهي فعلاً الأقوى لأنها بتستغل نقطة تانية في الـ URL parsing، وهي الفرق بين **fragment (`#`)** و**host**).

خلينا نفهمها خطوة بخطوة:

#### 1. Baseline request

تفتح منتج، تدوس "Check stock"، تعترض الـ request بـ Burp، وتبعته على Repeater. هتلاقي:

```
stockApi=http://stock.weliketoshop.net/product/stock/check?productId=1&storeId=1
```

#### 2. تجربة `127.0.0.1` مباشرة

```
stockApi=http://127.0.0.1/
```

هترفض. ده بيثبتلك إن السيرفر:

- بيعمل **parse** للـ URL
- بيطلع منه الـ **hostname**
- بيقارنه بـ **whitelist** (لازم يكون `stock.weliketoshop.net`)

#### 3. تجربة الـ `@` trick (embedded credentials)

```
stockApi=http://username@stock.weliketoshop.net/
```

**بيتقبل**! ده معناه إن الـ URL parser بتاع السيرفر بيفهم صيغة الـ **userinfo** (`username@host`)، وبيطلع الـ host الصح (`stock.weliketoshop.net`) من بعد الـ `@`، فالـ whitelist check بينجح عادي.

ده بيأكد إن السيرفر عنده parser "ذكي" شوية (مش بيتقبل بـ `@` قبل الـ domain المسموح زي الـ lab اللي فات، لأنه فاهم إن اللي قبل الـ `@` هو الـ credentials مش الـ host).

#### 4. إضافة `#` بعد الـ username

```
stockApi=http://username#@stock.weliketoshop.net/
```

**بترفض**! ليه؟ لأن الـ `#` في أي URL معناها بداية الـ **fragment** (زي الـ anchor في المتصفح). فالـ parser دلوقتي بيفهم الـ URL كده:

- **userinfo**: `username`
- **fragment**: `@stock.weliketoshop.net/`
- والـ **host فعليًا بقى فاضي أو مش موجود** (أو الـ URL بقى ناقص host صريح)

فالـ whitelist check بيفشل، لأن مفيش hostname واضح يتقارن بالـ whitelist.

#### 5. الخطوة الحاسمة: Double URL-encode للـ `#` → `%2523`

```
stockApi=http://username%2523@stock.weliketoshop.net/
```

هنا بيحصل حاجة مهمة جدًا:

- `%23` هو الـ URL encoding العادي لـ `#`
- `%2523` هو **double encoding**: `%25` = `%`, فلما تفك التشفير مرة واحدة بتوصل لـ `%23`

**السبب في الـ "Internal Server Error"**: السيرفر بيعمل عمليتين مختلفتين على نفس الـ string:

1. الـ **validation/whitelist check** بيحصل على الـ string وهو لسه فيه `%2523` (يعني مش متفكوك)، فالـ parser بيشوفه كـ **جزء عادي من الـ username** (مش fragment، لأن الـ `#` الحقيقي مشفّر مرتين)، فبيشوف الـ host = `stock.weliketoshop.net` → **يعدي الفحص** ✓
2. لكن لما السيرفر بيروح فعليًا **يبعت الـ HTTP request**، بيعمل **decode مرة واحدة** للـ URL (زي أي HTTP client عادي)، فـ `%2523` بترجع لـ `%23` (يعني `#` لسه مشفّرة مرة واحدة، مش الحرف الفعلي)...

الظاهرة اللي بتحصل فعليًا (اللي بتطلع "Internal Server Error"): بعد الـ decode الجزئي، الـ networking library بتاعة السيرفر (اللي بتبعت الـ request الفعلي) بتفهم الـ URL بطريقة مختلفة عن الـ validator، وبتحاول فعليًا تتصل بـ `username` كـ **hostname مستقل** (بعد ما الـ `%2523` اتحولت لـ `%23` واتفسرت كـ separator جزئي في مرحلة الاتصال)، فبتفشل التوصيلة وترجع Internal Server Error. الخطأ ده هو الدليل إن فيه **اختلاف تفسير (parsing discrepancy)** بين مرحلة الـ validation ومرحلة الـ execution.

#### 6. الاستغلال النهائي

بما إننا عارفين إن السيرفر (وقت التنفيذ الفعلي) بيقدر يفهم إن اللي قبل الـ `%2523@` هو الـ host الحقيقي، نستخدم بورت الـ admin بدل `username`:

```
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

يعني:

- الـ **validator** بيشوف الـ string ويلاقي `stock.weliketoshop.net` هو الـ host (بعد الـ `@`) → **يعدي** ✓
- الـ **HTTP client الفعلي** بيبعت الطلب لـ `localhost:80` (لأن بعد فك التشفير الجزئي، بيفهم إن ده هو الـ host:port الحقيقي، والـ `%23` بعد كده بتتحول أو بتتفسّر كنهاية للـ authority part)
- النتيجة: الطلب بيوصل فعليًا لـ `http://localhost:80/admin/delete?username=carlos` — وهو endpoint إداري بيحذف اليوزر `carlos`، والـ lab يتحل.

### الفرق بين الطريقتين (اللي شرحتهولك قبل كده والطريقة دي)

|  | `@` trick بسيط | `#` + double encoding |
| --- | --- | --- |
| بيستغل | فرق فهم الـ validator عن الـ parser للـ userinfo | فرق فهم الـ validator عن الـ execution client للـ **fragment + encoding layers** |
| بيشتغل مع | whitelist بسيطة بتستخدم regex/string matching | whitelist بتستخدم URL parser فعلي، لكن بتقارن *قبل* الـ full decoding |
| درجة التعقيد | بسيطة | متقدمة، وبتظهر بگ حقيقي في تعدد مراحل التعامل مع نفس الـ string |

### فايدة الـ Lab (الفكرة العامة اللي إنت لسه بتسألني عنها)

- بيوضح إن حتى لو الـ whitelist **بتستخدم URL parser حقيقي** (مش regex ساذج)، لسه ممكن تتكسر لو:
    - فيه **أكتر من مرحلة معالجة** للـ URL (مرة وقت الـ validation، ومرة وقت الـ execution)، وكل مرحلة بتفك الـ encoding بطريقة/عدد مرات مختلف
    - أو لو الـ parser مش متعامل صح مع أجزاء الـ URL المعقدة زي **userinfo** و**fragment**
- بيعلّمك مبدأ أمني أساسي: **"Parser Differential" / "Parsing Confusion"** — وهي فئة كاملة من الثغرات بتحصل لما مكونين مختلفين (أو نفس المكوّن في مرحلتين) بيفهموا نفس الـ input بطريقتين مختلفتين. الفئة دي مش بس في SSRF، لكن كمان في: Request Smuggling, Authentication Bypass, Path Traversal, Open Redirect.
- الحل الصح كمدافع: **استخدم نفس الـ parsed object** (نفس الـ URL object اللي طلع من نفس عملية الـ decode/parse) في كل من الـ validation والـ execution — متعملش parse مرتين بمنطق مختلف، ومتعتمدش على string comparison على أي جزء من الـ URL خام قبل ما يتفك بالكامل.

```bash
1. stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin

2. stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

### الـ Lab ده (SSRF with whitelist-based input filter)

هنا الحماية بقت أقوى من الـ blacklist: السيرفر بيستخدم **whitelist** — يعني مش بيرفض كلمات معينة، لكن بيتأكد إن الـ URL اللي جاي منك **لازم يتطابق (أو يبدأ بـ) domain محدد مسموح بيه فقط** (زي `stock.weliketoshop.net`). أي حاجة غير كده بترفض فورًا.

#### الفكرة

المشكلة إن كتير من تطبيقات الـ whitelist بتعتمد على **parsing ساذج للـ URL** — يعني بتفحص هل الـ string فيه الـ domain المسموح في مكان معين، من غير ما تفهم فعليًا إزاي الـ URL parser (بتاع اللغة اللي مكتوب بيها الكود) بيفسّر الأجزاء المختلفة زي الـ **scheme**, **userinfo**, **host**, **port**, **path**. الفرق بين "فهم الـ developer للـ URL" و"فهم الـ parser الفعلي" هو ثغرة أمنية بحد ذاتها.

#### إزاي بتتحل

الطريقة الأساسية هنا هي استغلال جزء الـ **userinfo** في الـ URL (اللي هو الجزء قبل `@`):

```
scheme://userinfo@host:port/path
```

كتير من الـ developers بيكتبوا فحص زي: "هل الـ URL يبدأ بـ `https://stock.weliketoshop.net`؟" — فلو حطينا الـ domain المسموح **قبل الـ `@`**، الفحص بيعدي (لأنه شايف الـ string المطلوبة في أول الـ URL)، لكن الـ **browser/HTTP client الفعلي بيتعامل مع اللي بعد الـ `@` باعتباره الـ host الحقيقي**:

```
stockApi=http://stock.weliketoshop.net@localhost/admin
```

هنا:

- الفحص (الـ regex أو الـ startsWith check) بيشوف `stock.weliketoshop.net` في الأول → **يعدي** ✓
- لكن فعليًا، `stock.weliketoshop.net` بقت مجرد **username** (userinfo)، والـ host الحقيقي اللي هيتبعتله الـ request هو `localhost`

#### خطوات الحل بالتفصيل

1. افتح صفحة منتج والقط الـ request بتاع "Check stock" في Burp.
2. جرّب الأول تتأكد من الفحص عن طريق تجربة قيمة عادية زي `http://localhost/admin` → هيترفض (blocked / not allowed domain).
3. عدّل قيمة الـ `stockApi` باستخدام الـ `@` trick:

```
stockApi=http://stock.weliketoshop.net@localhost/admin
```

1. ابعت الـ request — المفروض ترجع لك صفحة الـ admin panel في الـ response (بما إن الفحص عدّى والـ request فعليًا راح لـ `localhost/admin`).
2. من صفحة الـ admin، خد رابط زي `/admin/delete?username=carlos`، وحطه بنفس الـ trick:

```
stockApi=http://stock.weliketoshop.net@localhost/admin/delete?username=carlos
```

1. ابعت الـ request، ولو نجح، الـ lab يتحل.

#### فايدته إيه

- بيوضح إن **whitelist-based filtering** أقوى من الـ blacklist، لكنها برضو مش كافية لو مبنية على **string matching بسيط** بدل ما تكون مبنية على **parsing صحيح للـ URL** بنفس المنطق اللي هيستخدمه الـ HTTP client فعليًا وقت الإرسال.
- بيعلّمك جزئية مهمة جدًا في أمان الـ URLs: الفرق بين إزاي الـ **validation code** بيفهم الـ URL وإزاي الـ **execution code** (اللي بيبعت الـ request فعليًا) بيفهمه — أي اختلاف بين الاتنين ده هو ثغرة.
- بيرسّخ المبدأ الصحيح للدفاع: التحقق لازم يتم على **الـ host الفعلي بعد الـ parsing الكامل للـ URL** (باستخدام library موثوقة زي `URL` class في اللغة المستخدمة)، مش على الـ string الخام، وبيفضّل كمان استخدام **allow-list على مستوى الشبكة (network-level)** بدل ما يكون كله على مستوى الكود.
- بيفتح الباب لفهم أعمق في هجمات الـ **URL parsing confusion** اللي بتظهر في ثغرات تانية غير SSRF كمان (زي Open Redirect, Authentication Bypass, CORS misconfig).
