# Lab : 2FA broken logic

https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic

- الفكره من الLab ده ان ممكن تطخطي صفحه تسجيل الدخول وتروح لصفحه ال2FA علطول وتعدل فيها وتنسبها لtarget عن طريق تغير الverifity ل targrt  و بعدها تعمل brute force علي الكود

```bash
1. 
```

اللاب ده من فئة **Multi-Factor Authentication vulnerabilities**، وتحديدًا **خلل منطقي (Logic Flaw)** في آلية التحقق بخطوتين (2FA). المشكلة إن التطبيق بيحدد **الحساب اللي بيتم التحقق منه (verify) بناءً على قيمة parameter بيتحكم فيها المستخدم نفسه في الـ request**، مش بناءً على الـ session المرتبطة بمحاولة تسجيل الدخول الأصلية.

### آلية الـ Login في اللاب ده (Flow الطبيعي)

العملية بتتم على مرحلتين:

1. **Step 1**: تدخل username + password → لو صح، السيرفر بيبعتلك على الإيميل **كود تحقق (2FA code)**، وبيوديك لصفحة إدخال الكود، والـ URL بيكون شبه:

```
   GET /login2?verify=wiener
```

1. **Step 2**: تدخل الكود في فورم، والريكوست بيتبعت كده:

```
   POST /login2

   verify=wiener&mfa-code=1234
```

### المشكلة (الـ Logic Flaw)

الـ parameter اسمه `verify`، وقيمته هي **اسم المستخدم اللي بيتم التحقق منه في خطوة الـ 2FA**. المفروض إن القيمة دي تتحدد **تلقائيًا من السيرفر** بناءً على مين اللي دخل username/password صح في الخطوة الأولى (ومربوطة بالـ session بتاعته).

لكن هنا، القيمة دي **بتتبعت كـ parameter عادي في الـ request**، وأنت كمستخدم تقدر **تغيّرها بحرية** — والسيرفر بيثق فيها من غير أي ربط حقيقي بالـ session!

يعني لو غيّرت قيمة `verify` من `wiener` لـ `carlos`، السيرفر هيولّد كود تحقق (2FA code) خاص بـ **carlos**، وهيديك الفرصة تجرب تخمنه — حتى لو انت أصلاً سجلت دخول بحسابك انت (`wiener`) مش بحساب `carlos`.

### خطوات الحل بالتفصيل

#### 1. افهم آلية الـ 2FA بحسابك انت الأول

سجل دخول بـ:

```
wiener:peter
```

لاحظ في الـ `POST /login2` إن فيه parameter اسمه `verify` بيحمل قيمة `wiener` — وده اللي بيحدد صاحب الحساب اللي بيتعمله verify.

#### 2. اعمل Logout

سجل خروج من حسابك عشان تبدأ محاولة جديدة.

#### 3. ولّد كود 2FA لحساب carlos عن طريق التلاعب في الـ verify

ابعت الريكوست بتاع الصفحة (`GET /login2`) على **Burp Repeater**، وغيّر قيمة `verify` من `wiener` لـ:

```
GET /login2?verify=carlos HTTP/1.1
```

ابعت الريكوست ده. ده بيخلي السيرفر يولّد **كود 2FA مؤقت خاص بحساب carlos** (ويبعته على إيميله هو طبعًا، مش هتعرف تشوفه)، لكن المهم إنك خليت السيرفر "يفتح جلسة تحقق" باسم `carlos`.

#### 4. ادخل login عادي بحسابك أنت وابعت كود غلط

روح لصفحة تسجيل الدخول، ادخل:

```
username: wiener
password: peter
```

وبعد ما توصل لصفحة إدخال الكود، **ادخل أي كود غلط** (زي `0000`) وابعته، عشان تاخد الريكوست بتاع `POST /login2` كامل الشكل (فيه الـ cookie/session بتاعتك الصح).

#### 5. ابعت الريكوست على Burp Intruder

خد الـ `POST /login2` ده وابعته على **Intruder**:

```
POST /login2 HTTP/1.1
Cookie: session=<your-session>

verify=wiener&mfa-code=0000
```

#### 6. غيّر verify لـ carlos وحط payload على mfa-code

في الـ Intruder:

- غيّر قيمة `verify` يدويًا (ثابتة، مش payload) من `wiener` لـ `carlos`.
- حط **payload position** على قيمة `mfa-code` بس.
- استخدم قائمة أرقام (زي من `0000` لـ `9999`، أو استخدم الـ Numbers payload type في Burp) عشان تجرب كل الاحتمالات (الكود غالبًا 4 أرقام).

#### 7. شغّل الهجوم (Brute-force الكود)

شغّل الـ attack، وفلتر النتائج بحيث تدور على أي response مختلف عن الباقي (زي status code `302` بدل `200`، أو طول مختلف). لما تلاقيه، يبقى ده الكود الصحيح بتاع `carlos`.

#### 8. حمّل الـ response دي في المتصفح

ممكن تستخدم **"Request in browser"** في Burp أو تاخد الـ session/response وتحمّلها.

#### 9. افتح My Account

لو نجح، هتلاقي نفسك دخلت بحساب `carlos` رغم إنك سجلت دخول أصلاً بـ `wiener`! اضغط **"My account"** واللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
@app.route("/login2", methods=["GET"])def show_2fa_page():    username = request.args.get("verify")   # ❌ بيتاخد من الـ query parameter، مش من الـ session    generate_and_send_2fa_code(username)    return render_2fa_page(username)@app.route("/login2", methods=["POST"])def verify_2fa():    username = request.form.get("verify")   # ❌ نفس المشكلة هنا كمان    code = request.form.get("mfa-code")    if check_2fa_code(username, code):        login_user(username)   # بيسجل دخول بأي username اتحط في verify!        return redirect("/my-account")    return "Invalid code", 200
```

المشكلة الجوهرية: **السيرفر بيحدد "مين بيتعمله verify" من قيمة بيبعتها العميل نفسه في كل مرة، مش من حالة session ثابتة اتحددت وقت إدخال username/password الأول**. ده بيخلي الخطوة التانية (2FA) **منفصلة تمامًا ومستقلة** عن الخطوة الأولى، فتقدر "تخلط" بين الاتنين — تدخل بحسابك في خطوة 1، وتحول التحقق لحساب تاني في خطوة 2.

### الدرس المستفاد

- في أي عملية **Multi-step authentication** (زي 2FA)، **هوية المستخدم اللي بيتم التحقق منه لازم تُحفظ في الـ session من الخطوة الأولى**، ومتاخدش تاني من أي parameter بيتحكم فيه العميل في الخطوات اللاحقة.
- لو أي خطوة في عملية الـ auth بتقبل معرف هوية (username, user_id) كـ parameter صريح بدل ما تعتمد على السياق المحفوظ (session)، ده بيفتح باب لخلط الهويات بين المستخدمين.
- **2FA code لازم يتولد ويترتبط بمحاولة تسجيل دخول واحدة محددة (tied to that specific session)**، مش بيتولد بشكل منفصل بناءً على طلب مستقل ممكن يتلاعب فيه أي حد.
- ده مثال ممتاز على إزاي إضافة طبقة أمان زيادة (2FA) **من غير تصميم سليم** ممكن تفتح ثغرة جديدة بدل ما تسد ثغرة قديمة.
