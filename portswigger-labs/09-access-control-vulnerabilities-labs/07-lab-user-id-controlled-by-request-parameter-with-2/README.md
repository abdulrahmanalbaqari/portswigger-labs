# Lab : User ID controlled by request parameter with password disclosure

https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure

- الفكره من الLab ده ان في parameter اسمه Upgrade-Insecure-Requests مصاب من غير حمايه اول مغير القيم بتاعته واعدل علي الid ل administrator بكتسب صلاحياته او الpassword بتاعه واقدر a access  عليه بدون اي مشاكل

```bash
1. GET /my-account?id=administrator

2. Upgrade-Insecure-Requests: 2
```

ده لاب تاني من فئة **Broken Access Control**، وتحديدًا **Insecure Direct Object Reference (IDOR)** — نفس عيلة اللاب اللي فات، بس المشكلة هنا مش بس إنك تقدر تشوف بيانات حساب مستخدم تاني، لكن الـ endpoint نفسه **بيسرب الباسورد** بتاعه كمان، وده بيخليك تقدر تستولي على الحساب بالكامل (Account Takeover)، مش مجرد تشوف بياناته.

### طبيعة الثغرة في اللاب ده

التطبيق فيه صفحة (غالبًا "Update email" أو "My account") بترجع تفاصيل المستخدم بناءً على `id` بيتبعت كـ **query parameter**، زي:

```
GET /my-account?id=wiener
```

المشكلة إن الـ **response** بتاعة الصفحة دي (سواء في الـ HTML نفسه، أو خصوصًا في الـ **API request** اللي بتحصل في الخلفية عشان تجيب بيانات اليوزر) بترجع الباسورد بشكل **plaintext** أو مخبأ جوه الـ HTML (زي في input field من نوع hidden أو مكتوب كـ value في حقل الباسورد).

### خطوات الحل

1. **سجل دخول** بحسابك (`wiener:peter`).
2. روح لصفحة **"My account"**، وافتح Burp Suite واعمل انترسبت للريكوستات اللي بتحصل.
3. لاحظ إن فيه ريكوست بيتبعت شبه:

```
   GET /my-account?id=wiener HTTP/1.1
```

وده اللي بيجيب بيانات الحساب (زي الإيميل والباسورد المعروض في الفورم).

1. **افحص الـ Response** بتاع الريكوست ده كويس — هتلاقي إن فيه حقل مخفي أو قيمة في الـ HTML فيها الباسورد الحالي لليوزر ده، حتى لو مش ظاهر بصريًا في الصفحة (ممكن يكون input type="hidden" أو داخل الـ HTML مباشرة).
2. **غيّر قيمة الـ `id`** في الريكوست عشان تستهدف مستخدم تاني (زي `administrator` أو `carlos`)، وابعته من Repeater:

```
   GET /my-account?id=administrator HTTP/1.1
```

1. **افحص الـ Response تاني**، لو الثغرة موجودة هتلاقي **باسورد الـ administrator** ظاهر في الـ HTML response (زي `value="admin-password-here"` في حقل باسورد مخفي).
2. **انسخ الباسورد** ده.
3. **اعمل Logout**، وسجل دخول تاني باستخدام:

```
   username: administrator
   password: <الباسورد اللي لقيته>
```

1. لو دخلت بنجاح كـ administrator، يبقى اللاب اتحل ✅ (عادةً PortSwigger بيتأكد من الحل بمجرد دخولك كـ admin بنجاح).

### ليه ده بيحصل تقنيًا

الكود بيكون شبه:

python

```python
user_id = request.get("id")user = database.get_user(user_id)# مفيش تحقق إن اليوزر الحالي (من الـ session) هو نفسه user_id المطلوبreturn render_template("account.html",                        email=user.email,                        password=user.password)  # ❌ بيرجع الباسورد في الـ response نفسه
```

المشكلة هنا مركبة من حاجتين:

1. **مفيش access control** — أي حد يقدر يبعت أي `id` ويجيب بيانات أي مستخدم تاني (زي اللاب السابق).
2. **تسريب بيانات حساسة زيادة عن اللزوم (Excessive Data Exposure)** — السيرفر أصلاً بيرجع الباسورد في الـ response حتى لو مطلوب بس عرض الإيميل، وده خطأ في تصميم الـ API/endpoint بغض النظر عن مشكلة الـ IDOR.

### الدرس المستفاد

- لازم أي endpoint بيرجع بيانات حساب، **يتحقق من هوية الطالب** من الـ session/cookie، مش من قيمة بيبعتها المستخدم نفسه في الطلب.
- **مبدأ الحد الأدنى من البيانات (Data Minimization)**: السيرفر ميرجعش أي بيانات حساسة (زي الباسورد، حتى لو hashed) في أي response إلا لو محتاجها فعليًا العميل، وحتى لو محتاجها، الباسورد بالذات ميفترضش إنه يترجع للـ frontend خالص في أي حالة طبيعية.
- الثغرة دي بتوضح إزاي مشكلة IDOR بسيطة ممكن تتحول من "تسريب معلومات" لـ "استيلاء كامل على حساب" (Account Takeover) لو اتضافلها تسريب بيانات حساسة زي الباسورد.
