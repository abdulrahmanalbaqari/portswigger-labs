# Lab : Brute - Force a stay - logged - in cookie

https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie

- الفكره من الLab ده ان ال cookie فيها parameter مصاب stay-logged-in  فا دي ممكن تخلك login منغير اي حاجه بس فيها مش كله انك المفروض تعمل brute force بس هي معموله بطبقه من التشفير base64 and hash MD5  فا دي حلها في الburp suite في intruder payload process

```bash
1. payload processing :
	Hash:MD5
	AddPrefix:carlos:
	Base64-encode
	
	
2.  stay-logged-in=Y2FybG9zOjliMzA2YWIwNGVmNWUyNWY5ZmI4OWM5OThhNmFlZGFi

3. carlos:9b306ab04ef5e25f9fb89c998a6aedab

4. carlos:george

5. GET /my-account HTTP/2
Host: 0afa00780359904f84935a2d00db0020.web-security-academy.net
Cookie: session=PmqGeQsoMpFlPbmRTfVrcrv8aDjxzVdd; stay-logged-in=Y2FybG9zOjliMzA2YWIwNGVmNWUyNWY5ZmI4OWM5OThhNmFlZGFi
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0afa00780359904f84935a2d00db0020.web-security-academy.net/my-account?id=wiener
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

```

اللاب ده من فئة **Other authentication mechanisms**، وتحديدًا مشكلة في تصميم كوكي **"Stay logged in"** (تذكرني / ابقيني مسجل دخول). الفكرة إن الكوكي ده مبني بطريقة **يمكن التنبؤ بيها وإعادة بنائها (predictable/reconstructible)**، لأنه مبني من بيانات معروفة أو سهلة التخمين، بدل ما يكون token عشوائي طويل (زي UUID أو random string) محفوظ في قاعدة البيانات ومربوط بجلسة معينة.

### طبيعة الثغرة

لما تختار **"Stay logged in"** وقت تسجيل الدخول، السيرفر بيولّدلك كوكي اسمه `stay-logged-in`، قيمته بتبقى **Base64-encoded**. لو فككتها (decode)، هتلاقي شكلها:

```
wiener:51dc30ddc473d43a6011e9ebba6ca770
```

يعني الصيغة هي:

```
username : hash_of_something
```

الجزء التاني طوله وشكله (32 حرف hex) بيدل على إنه **MD5 hash**. وبما إن المعروف هو الـ username، الافتراض المنطقي إن ده هو `MD5(password)`. لو عملت hash لباسوردك انت بـ MD5، هتلاقيه مطابق تمامًا. يبقى الصيغة الكاملة:

```
Cookie = Base64( username + ":" + MD5(password) )
```

**المشكلة**: الكوكي ده **مش عشوائي (not a random token)** — هو **دالة رياضية بسيطة** من حاجتين تقدر تعرفهم أو تخمنهم: الـ **username** (معروف) والـ **password** (تقدر تخمنه من قائمة باسوردات شائعة). يعني تقدر **تبني الكوكي بنفسك** من غير ما تحتاج تسرقه، لو عرفت تخمن الباسورد.

### خطوات الحل بالتفصيل

#### المرحلة 1: فهم بنية الكوكي

1. سجل دخول بحسابك (`wiener:peter`) واختار **"Stay logged in"**.
2. افحص الكوكي `stay-logged-in` في **Burp Inspector** (بيفك الـ Base64 تلقائيًا).
3. هتلاقي القيمة: `wiener:51dc30ddc473d43a6011e9ebba6ca770`.
4. اعمل MD5 لكلمة `peter` (باسوردك) — هتلاقيها نفس الـ hash بالظبط. يبقى تأكدنا إن الصيغة هي:

```
   Base64(username + ":" + MD5(password))
```

#### المرحلة 2: تجربة الطريقة على حسابك انت (للتأكد)

1. اعمل **Logout**.
2. خد آخر ريكوست `GET /my-account?id=wiener` (اللي فيه الكوكي القديم)، وابعته على **Burp Intruder**.
3. لاحظ إن Burp حط تلقائيًا **payload position** على قيمة `stay-logged-in` cookie.
4. في الـ **Payloads panel**: ضيف **باسوردك (`peter`) كـ payload واحد بس** (عشان نتأكد الطريقة شغالة الأول).
5. تحت **Payload processing**، ضيف القواعد دي **بالترتيب ده بالظبط**:

يعني كل payload بيمر بمراحل: `peter` → `MD5(peter)` → `wiener:MD5(peter)` → `Base64(wiener:MD5(peter))`، وده بالظبط نفس طريقة تكوين الكوكي الأصلي.
    - **Hash**: `MD5` → يحول الباسورد لـ MD5 hash.
    - **Add prefix**: `wiener:` → يضيف اليوزرنيم وعلامة `:` قبل الـ hash.
    - **Encode**: `Base64-encode` → يحول الناتج النهائي لـ Base64.
6. بما إن زرار **"Update email"** بيظهر بس لو انت **مسجل دخول فعليًا**، نستخدم وجوده كمؤشر نجاح. في **Settings panel**، ضيف **Grep - Match rule** يدور على النص:

```
    Update email
```

1. شغّل الهجوم (Start attack).

#### المرحلة 3: التأكد من صحة الطريقة

1. لاحظ إن الـ response للـ payload الوحيد (باسوردك) فعلاً ظهر فيه "Update email" — يعني الكوكي اللي بنيناه يدويًا نجح في تسجيل الدخول بحسابك انت. ده بيأكد إن الـ payload processing rules شغالة صح 100%.

#### المرحلة 4: تنفيذ الهجوم الفعلي على carlos

1. اعمل التعديلات دي وكرر الهجوم:
    - **شيل باسوردك** من قائمة الـ payloads، وحط بدالها **قائمة الباسوردات المرشحة (candidate passwords)**.
    - في الـ URL، غيّر الـ `id` parameter من `wiener` لـ `carlos`:

```
      GET /my-account?id=carlos
```

```
- غيّر قاعدة **Add prefix** من `wiener:` لـ `carlos:`.
```

14. شغّل الهجوم من جديد.

1. لما يخلص، **افحص عمود الـ Grep-Match**: هتلاقي **response واحد بس** فيه `Update email` — ده معناه إن الباسورد المقابل له هو باسورد `carlos` الصحيح، والـ payload بتاعه (بعد المعالجة) هو الكوكي الصحيح بتاعه.
2. **خد الكوكي ده** (القيمة النهائية بعد الـ Base64 encoding)، وحطه كـ `stay-logged-in` cookie في طلبك، وافتح صفحة `/my-account?id=carlos`.
3. لو ظهرت صفحة حسابه بنجاح، اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
def generate_stay_logged_in_cookie(username, password):    hash_value = md5(password)    raw = f"{username}:{hash_value}"    return base64_encode(raw)def verify_cookie(cookie_value):    decoded = base64_decode(cookie_value)    username, password_hash = decoded.split(":")    stored_hash = md5(get_password(username))    if password_hash == stored_hash:        login_user(username)
```

المشكلة الجوهرية: الكوكي **مش token عشوائي غير قابل للتخمين** مُخزّن في قاعدة بيانات ومرتبط بجلسة، لكنه **دالة حتمية (deterministic function)** من بيانتين: username (معروف للجميع) وMD5(password) (باسوردات كتير من المستخدمين بتكون من قوائم شائعة سهلة التخمين). فبما إن الـ MD5 نفسه سريع الحساب جدًا (مصمم للسرعة مش للأمان زي bcrypt)، أي حد يقدر:

1. يجرب كل باسورد من قائمة شائعة.
2. يعمل MD5 لكل واحد.
3. يبني الكوكي المكافئ.
4. يجربه على السيرفر.

### الدرس المستفاد

- **كوكيز "Remember me" / "Stay logged in" لازم تكون tokens عشوائية طويلة (cryptographically random)** مُخزّنة في قاعدة البيانات (أو موقّعة بمفتاح سري قوي على السيرفر)، **مش مبنية من بيانات يمكن حسابها أو تخمينها** (زي username + hash بسيط للباسورد).
- **MD5 غير آمن لتخزين أو اشتقاق باسوردات** — سريع جدًا في الحساب، وده بالظبط اللي بيخلي الـ brute-force عملي وسريع. المفروض استخدام دوال مصممة عمدًا لتكون بطيئة زي **bcrypt, scrypt, أو Argon2**.
- أي بيانة **يقدر المستخدم يفكها (decode)** زي هنا Base64 لازم متفترضش إنها "مخفية" — الـ encoding مش تشفير (encryption)، وأي حد يقدر يفك أي Base64 في ثانية.
- **Payload Processing Rules في Burp Intruder** (Hash → Add prefix → Encode) أداة قوية جدًا لما تحتاج تعيد بناء قيمة معقدة (زي هنا) بشكل ديناميكي لكل payload، بدل ما تحسبها يدويًا لكل باسورد على حدة.
- استخدام **Grep - Match** لتحديد نجاح الهجوم (زي وجود كلمة "Update email") تكنيك أساسي ومتكرر في اختبار الاختراق الآلي، خصوصًا لما status code وحده مش كافي للتمييز بين النجاح والفشل.
