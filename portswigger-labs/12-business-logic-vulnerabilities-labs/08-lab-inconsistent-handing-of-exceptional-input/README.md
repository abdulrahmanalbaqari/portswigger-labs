# Lab : Inconsistent handing of exceptional input

https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-inconsistent-handling-of-exceptional-input

- الفكره من الLab ده ان انت عاوز تاخد صلاحيات admin بس ده مش هيحصل الا لو الemail بتاعك اخره @dontwannacry.com بس ده تبع الtarget واذاي هعمل verification وانا مش معايا الemail بتاعه فا هطر احوله لsubdomain من الemail بتاعي بعد كده هستغل الbug  ان الموقع مش بيراجع ورايا وهخل الname بتاع الemail very long string علشان الموقع ليه limit ويمسح domain بتاعي ويسيب الtarget

```bash
1. Your email is: test@dontwannacry.com.exploit-0ac1001303720d6a80a4d43701be00e0.exploit-server.net

2. email=sssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss222222222222222222222%40dontwannacry.com.exploit-0ac1001303720d6a80a4d43701be00e0.exploit-server.net

3. sssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss222222222222222222222%40dontwannacry.com ====> هيتقرا كده 
```

اللاب ده من فئة **Business Logic Vulnerabilities** (اللي اتكلمنا عنها قبل كده)، وتحديدًا **Logic Flaw** بسبب **عدم التحقق الكافي من طول/شكل الـ input** (Exceptional/Boundary input handling).

الفكرة الأساسية: التطبيق فيه **حقلين مختلفين بيتعاملوا مع نفس البيانات (الإيميل) بطريقتين مختلفتين**:

1. **نظام إرسال الإيميلات** — بيقبل الإيميل كامل بأي طول.
2. **قاعدة البيانات / الحقل المخزّن للإيميل** — بيقصّ (truncate) أي إيميل أطول من **255 حرف**.

التناقض ده بين الطبقتين هو أساس الثغرة.

### طبيعة الاستغلال (الفكرة الذكية)

#### السياق

الموقع بيديك صلاحيات إدارية **تلقائيًا** لو سجّلت بإيميل ينتهي بـ `@dontwannacry.com` (دومين الشركة الداخلي).

#### الخطة

لو قدرت **تخلي إيميلك يبان وكأنه بينتهي بـ `@dontwannacry.com`**، لكن في الحقيقة هو إيميل مختلف بتتحكم فيه انت (عشان تقدر تستلم إيميل التأكيد)، تقدر تخدع النظام.

#### إزاي؟ باستخدام الـ subdomain trick + الـ truncation

لو كتبت إيميل زي:

```
xxxxxxxxxx@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net
```

الإيميل ده **فعليًا وقانونيًا صحيح** وبيوصل لسيرفر الإيميل بتاعك انت (لأن الدومين الحقيقي هو `YOUR-EMAIL-ID.web-security-academy.net`، و`dontwannacry.com` هنا مجرد subdomain وهمي جواه). فالسيرفر هيقدر يبعتلك إيميل التأكيد عادي.

**لكن**: لو الإيميل ده طوله أكتر من 255 حرف، والتطبيق بيقص الإيميل عند الحرف 255 بالظبط لما يخزّنه في قاعدة البيانات — لو حسبنا طول الـ `very-long-string` بدقة بحيث **حرف الـ 255 يقع بالظبط عند آخر حرف في `@dontwannacry.com`** (يعني عند حرف الـ `m`)، النتيجة إن القيمة **المخزّنة في قاعدة البيانات** هتبقى:

```
xxxxxxxxxx@dontwannacry.com
```

(القصّ هيشيل باقي الدومين الحقيقي `.YOUR-EMAIL-ID.web-security-academy.net` بالكامل!)

يعني: **الإيميل الحقيقي بيوصلك انت** (عشان السيرفر لسه بيستخدم النسخة الكاملة وقت الإرسال)، لكن **الإيميل المخزّن في حسابك** بيبقى شكله زي إيميل شرعي من `dontwannacry.com` — وده اللي بيدّيك صلاحيات الأدمن.

### خطوات الحل بالتفصيل

#### 1. اكتشف صفحة `/admin` عن طريق Content Discovery

افتح اللاب وانت شغّل Burp Proxy. روح لـ **Target > Site map**، كليك يمين على دومين اللاب واختار:

```
Engagement tools → Discover content
```

دوس **"Session is not running"** لبدء الفحص. الأداة دي بتحاول تكتشف مسارات مخفية عن طريق تجربة أسماء شائعة. بعد شوية، هتلاقي في النتائج مسار:

```
/admin
```

#### 2. جرب تفتح `/admin`

هتلاقي رسالة خطأ بتقول إن المستخدمين اللي إيميلهم من **`DontWannaCry`** (الشركة) هم بس اللي عندهم صلاحية الوصول.

#### 3. روح لصفحة التسجيل (Registration)

هتلاقي رسالة موجهة لموظفي DontWannaCry تقولهم يستخدموا **إيميل الشركة الرسمي** وقت التسجيل.

#### 4. افتح الإيميل كلاينت من زرار في بانر اللاب

هتلاقي **الـ ID الفريد بتاعك** في اسم الدومين (`@YOUR-EMAIL-ID.web-security-academy.net`) — **سجّله**، هتحتاجه في الخطوات الجاية.

#### 5. سجّل حساب بإيميل طويل جدًا (تجربة أولى للفهم)

سجّل بإيميل شكله:

```
very-long-string@YOUR-EMAIL-ID.web-security-academy.net
```

حيث `very-long-string` طوله **200 حرف على الأقل** (أي نص عشوائي طويل).

#### 6. افتح الإيميل كلاينت وأكد التسجيل

هتلاقي إيميل تأكيد وصلك، دوس على اللينك بتاعه عشان تكمل التسجيل.

#### 7. سجل دخول وشوف صفحة My Account

هتلاحظ إن الإيميل المعروض في حسابك **اتقص (truncated) عند 255 حرف بالظبط**. ده **التأكيد العملي** إن التطبيق فعلاً بيقص الإيميل عند الحفظ في قاعدة البيانات، وده بالظبط الـ logic flaw اللي هنستغله.

#### 8. اعمل Logout وارجع لصفحة التسجيل من جديد

#### 9. سجل حساب جديد بإيميل مصمم بدقة

المرة دي، حط `dontwannacry.com` كـ **subdomain وهمي** جوه الإيميل الحقيقي بتاعك:

```
very-long-string@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net
```

**النقطة الحرجة**: لازم تحسب طول الـ `very-long-string` بدقة، بحيث **حرف الـ `m` في نهاية `dontwannacry.com`** يكون **بالظبط الحرف رقم 255** من بداية الإيميل. لو الحساب مظبوط، القص هيحصل **بالظبط بعد** `dontwannacry.com`، وهيشيل الباقي (`.YOUR-EMAIL-ID.web-security-academy.net`) بالكامل.

#### 10. افتح الإيميل كلاينت وأكد الحساب

هتلاقي إيميل التأكيد وصلك عادي (لأن الإرسال الفعلي بيستخدم **العنوان الكامل الحقيقي**، مش المقصوص). دوس على اللينك.

#### 11. سجل دخول وتأكد من النتيجة

روح لصفحة **My Account**، هتلاقي الإيميل المعروض دلوقتي **بينتهي بـ `@dontwannacry.com`** (بسبب القص)، وهتلاقي إن عندك دلوقتي **وصول لصفحة الأدمن بانل**.

#### 12. احذف carlos

روح للأدمن بانل واحذف `carlos`. اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
@app.route("/register", methods=["POST"])def register():    email = request.form.get("email")    send_confirmation_email(email)   # ✅ بيستخدم الإيميل الكامل من غير قص    db.execute("INSERT INTO users (email) VALUES (?)", email[:255])   # ❌ بيقص عند 255 حرف@app.route("/admin")def admin_panel():    user_email = get_current_user_email()   # القيمة المخزّنة (المقصوصة!)    if user_email.endswith("@dontwannacry.com"):        return render_admin_panel()    return "Unauthorized", 403
```

المشكلة الجوهرية: **حقل الإيميل بيتعامل معاه بطريقتين مختلفتين في مكانين مختلفين من التطبيق**:

- **عند الإرسال (sending)**: بيستخدم القيمة **الكاملة** بدون أي حد أقصى.
- **عند التخزين (storage)**: بيقصّ القيمة عند حد أقصى (255 حرف، وده الحد الشائع لحقول `VARCHAR` في قواعد بيانات كتير).

بما إن **فحص الصلاحية (`endswith dontwannacry.com`) بيتم على القيمة المخزّنة بعد القص**، مش على القيمة الأصلية الكاملة، قدرنا "نصمم" الإيميل بحيث القص نفسه يحوّله لإيميل شرعي الشكل.

### الدرس المستفاد

- **أي input معقد (زي إيميل) لازم يتحقق منه بنفس الطريقة والمعايير في كل نقطة في التطبيق تتعامل معاه** — مفيش مكان يقبل طول لا نهائي ومكان تاني بيقص، لأن التناقض ده نفسه بيبقى ثغرة.
- **التحقق من الصلاحيات لازم يحصل على البيانات "الأصلية" الصحيحة**، مش على نسخة معدّلة أو مقصوصة منها بعد أي معالجة (processing) داخلية.
- **حدود قواعد البيانات (زي `VARCHAR(255)`) لازم تتفرض من نقطة الإدخال الأولى (validation عند الإدخال)**، مش تُترك للقص التلقائي الصامت عند التخزين — لأن القص الصامت ده بيغيّر معنى البيانات من غير ما حد ينتبه.
- ده مثال ممتاز على **"Exceptional input handling"** — يعني إزاي التعامل مع **حالات حدّية/استثنائية (edge cases)** زي طول نص غير متوقع، ممكن يفتح باب لاستغلال لوجيكي خطير، حتى لو مفيش أي كود "مكسور" تقنيًا (مفيش SQL injection ولا XSS، الكود شغال "صح" لكن منطقه فيه ثغرة).
- أداة **"Discover content"** في Burp مفيدة جدًا لاكتشاف مسارات مخفية (زي `/admin`) مش مرتبط بيها من أي مكان ظاهر في الموقع.
