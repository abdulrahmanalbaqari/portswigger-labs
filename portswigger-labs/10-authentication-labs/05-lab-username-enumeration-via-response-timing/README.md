# Lab : Username enumeration via response timing

https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing

- الفكره من الLab ده ان الموقع ممكن تعمله brute force بس فيه شويه حمايه بس ممكن تعمله bypass بكل سهوله عنطريق X-Forwarded-For

```bash
1. X-Forwarded-For: 10.0.0.$$

2. POST /login HTTP/2
Host: 0a48004c044205d380ad7b780001000f.web-security-academy.net
Cookie: session=iKbwGtNq8vsCZF7B6VDvGgzW5YN5T2rf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
X-Forwarded-For: 10.0.0.
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Origin: https://0a48004c044205d380ad7b780001000f.web-security-academy.net
Referer: https://0a48004c044205d380ad7b780001000f.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

username=$$&password=1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111

3. POST /login HTTP/2
Host: 0a48004c044205d380ad7b780001000f.web-security-academy.net
Cookie: session=iKbwGtNq8vsCZF7B6VDvGgzW5YN5T2rf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
X-Forwarded-For: 10.0.0.
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Origin: https://0a48004c044205d380ad7b780001000f.web-security-academy.net
Referer: https://0a48004c044205d380ad7b780001000f.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

username=af&password=$$
```

### الفكرة الأساسية

اللاب فيه ثغرتين مجتمعين:

1. **IP-based rate limiting** ممكن تتجاوزه بـ `X-Forwarded-For` header (زي ما اتكلمنا)
2. **Timing side-channel**: السيرفر بيتأخر في الرد بمقدار مرتبط بطول الـ password، لكن **بس لو الـ username صحيح**. يعني الكود بيتحقق من الـ password حرف حرف (أو بطريقة بتاخد وقت متناسب مع الطول)، فلو الـ username غلط أصلاً بيرفض على طول من غير ما يدخل في مقارنة الباسورد.

يعني: زمن الاستجابة بيبقى مؤشر على "هل الـ username ده موجود ولا لأ" حتى من غير ما تعرف الباسورد.

### المرحلة الأولى: اكتشاف الـ username الصحيح

**الخطوات:**

1. ابعت الـ login request لـ Repeater، جرب username/password غلط، لاحظ إنك بعد كذا محاولة هتتحجب (IP blocked)
2. اكتشف إن `X-Forwarded-For` بيتقبل ويعمل bypass للحجب
3. جرب بحساب نفسك (username صح) بباسورد طويل ولاحظ إن الوقت بيزيد مع طول الباسورد، عكس لما يكون الـ username غلط (بيرد بسرعة ثابتة)

**بناء على كده، الاستراتيجية:**

- ثبّت الباسورد يكون **string طويل جداً (100 حرف)** لكل الـ usernames
- لو الـ username **غلط**: هيرفض بسرعة (زمن قليل وثابت)
- لو الـ username **صح**: السيرفر هيدخل يقارن الباسورد الطويل حرف حرف، وده هياخد وقت أطول ملحوظ

**Intruder setup (Pitchfork):**

- Position 1: `X-Forwarded-For: §1§` → Numbers 1-100 (كل attempt بـ IP مختلف عشان تتجاوز الـ rate limit)
- Position 2: `username=§2§` → list الـ usernames
- الـ password: ثابت = string طويل (100 حرف) مش marker

استخدمنا **Pitchfork** مش Cluster bomb لأننا عايزين كل IP (رقم) يتقابل مرة واحدة بس مع كل username بالتوازي (صف بصف)، مش كل التوليفات.

**بعد التنفيذ:**

- شغّل عمودي **Response received** و **Response completed** في النتايج (بيوضحوا زمن الاستجابة بالتفصيل)
- دوّر على الصف اللي زمنه أعلى بشكل ملحوظ عن الباقي → ده الـ username الصحيح (كرر الطلب كذا مرة تتأكد إن التأخير ثابت مش عشوائي)

### المرحلة الثانية: brute force على الباسورد بنفس التقنية

دلوقتي عندك الـ username الصح، فالباسورد الحقيقي طوله معروف تقريباً (لأن الوقت بيتناسب مع عدد الحروف الصح اللي اتقارنت... لكن هنا مش هنعتمد على ده، هنعمل brute force عادي)

**Intruder setup تاني (Pitchfork تاني):**

- Position 1: `X-Forwarded-For: §1§` → Numbers (لتفادي الحجب برضه)
- Position 2: `password=§2§` → list الباسوردات المحتملة
- الـ username: ثابت = اللي اكتشفناه فوق

**بعد التنفيذ:**

- دوّر في النتايج على أي response بحالة **302** (يعني تسجيل دخول ناجح، عادة بيعمل redirect)
- ده معناه الباسورد ده هو الصحيح

### آخر خطوة

سجل دخول بالـ username والـ password اللي لقيتهم، ادخل على صفحة الحساب → اللاب يتحل.

**ملخص ليه استخدمنا Pitchfork مرتين مش Cluster bomb:**

لأن في كل مرحلة إحنا مش عايزين نجرب كل التوليفات بين list1 و list2، إحنا بس عايزين كل قيمة IP تتزامن مع القيمة المقابلة لها في list التانية (سطر بسطر) — ده بالظبط تعريف Pitchfork، وبيوفر وقت وعدد requests أقل من Cluster bomb اللي كان هيضاعف العدد من غير داعي.
