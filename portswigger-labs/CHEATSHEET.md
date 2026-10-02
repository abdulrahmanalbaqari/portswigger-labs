# ملخص سريع لااغلب الlabs الي حلتها

# 📊 الجدول الشامل النهائي — كل اللابات اللي شرحناها

## 1️⃣ File Path Traversal

| اللاب | السبب الجذري | الـ Bypass / الدرس الأساسي |
| --- | --- | --- |
| Traversal sequences blocked with absolute path bypass | الفلتر بيشيل `../` بس، مش بيمنع absolute paths | استخدم `/etc/passwd` مباشرة بدل `../../../etc/passwd` |
| Traversal sequences stripped non-recursively | الفلتر بيشيل `../` مرة واحدة بس (مش loop) | `....//` → بعد الحذف تتكون `../` من جديد |
| Traversal sequences stripped with superfluous URL-decode | الفلترة بتحصل قبل decode | `..%2f..%2f..%2fetc/passwd` |
| Validation of start of path | فحص `startswith()` بس، بدون canonicalization | `/var/www/images/../../../etc/passwd` |
| Validation of file extension with null byte bypass | تناقض بين طبقة الفحص وطبقة فتح الملف | `../../../etc/passwd%00.png` |

**الدرس العام**: الفلترة الجزئية دايمًا قابلة للتخطي؛ الحل الصح هو **canonical path validation** بعد الـ resolve الكامل.

## 2️⃣ Access Control (Broken)

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| User ID controlled by param (unpredictable ID) | GUID صعب التخمين لكن بيتسرب من مكان تاني (reviews) | صعوبة التخمين ≠ access control حقيقي |
| User ID with password disclosure | الـ endpoint بيرجع الباسورد في الـ response | Data minimization + auth check لازم يكونوا مع بعض |
| Insecure direct object references | أرقام متسلسلة كأسماء ملفات (transcripts) | استخدم UUIDs + تحقق ownership دايمًا |
| URL-based access control circumvented | Front-end بيفحص path، back-end بيثق بـ `X-Original-URL` | لا تعتمد على front-end وحده للحماية |
| Method-based access control circumvented | فحص الصلاحية جوه `if method == POST` بس | افحص الصلاحية بغض النظر عن الـ method |
| Multi-step process, no access control on one step | فحص بس على الخطوة الأولى | كل endpoint بيغيّر حالة يحتاج فحصه الخاص |
| Referer-based access control | الثقة في هيدر `Referer` يتحكم فيه العميل | لا تستخدم headers قابلة للتزوير كدليل صلاحية |

## 3️⃣ Authentication

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| Broken brute-force protection, IP block | نجاح لوجين (أي حساب) بيصفّر العداد | الحظر لازم يكون على مستوى الحساب المستهدف |
| Username enumeration via account lock | رسالة القفل بتظهر بس لو اليوزر موجود | رسائل موحدة بغض النظر عن حالة الحساب |
| 2FA broken logic | `verify` parameter بيتحدد من العميل مش من الـ session | هوية اللي بيعمله verify لازم تتحفظ في session |
| Password brute-force via password change | رسالة مختلفة (`New passwords do not match`) بتسرّب صحة current-password | رسائل موحدة + rate limiting على current-password |
| Broken brute-force, multiple credentials per request | JSON array بدل string واحد للباسورد | validate نوع البيانات + عدّ محاولات فعلية مش requests |
| 2FA bypass using brute-force | كود 2FA 4 أرقام بس + logout بعد محاولتين | Burp Macros لأتمتة إعادة اللوجين + كود أطول |
| Brute-forcing stay-logged-in cookie | كوكي = `base64(user:md5(password))` | Cookies العشوائية لازم tokens عشوائية، مش دالة من الباسورد |
| Offline password cracking | نفس الفكرة + XSS لسرقة الكوكي | MD5 قابل للكسر بسرعة (rainbow tables) |
| Password reset poisoning via middleware | `X-Forwarded-Host` بيبني رابط الـ reset | استخدم domain ثابت لبناء روابط حساسة |

## 4️⃣ Insecure Deserialization

| المفهوم | الشرح |
| --- | --- |
| Object Injection | استبدال object بكلاس مختلف تمامًا متاح في التطبيق |
| Gadget Chains | سلسلة استدعاءات methods غير متوقعة تؤدي لـ RCE |
| الدرس الأساسي | تجنب deserialize لبيانات المستخدم؛ لو لازم، وقّع البيانات رقميًا قبل الـ deserialization |

## 5️⃣ Information Disclosure

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| Debug page | صفحة `phpinfo.php` منسية في production | امسح صفحات الـ debugging من production تمامًا |
| Authentication bypass via info disclosure | `TRACE` method كشفت اسم هيدر `X-Custom-IP-Authorization` | عطّل TRACE method؛ لا تثق بهيدرز IP قابلة للتزوير |
| Version control history | فولدر `.git/` مكشوف + باسورد قديم في commit history | أي سر ظهر في Git history يُعتبر محروق للأبد |

## 6️⃣ Business Logic Flaws

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| Inconsistent handling of exceptional input | Email طويل بيتقص عند 255 حرف عند التخزين، مش عند الإرسال | Validation والتخزين لازم يتفقوا على نفس الحدود |
| Authentication bypass via encryption oracle | نفس مفتاح التشفير لـ auth cookie و notification message | لا تستخدم نفس المفتاح لأغراض مختلفة؛ افهم حدود block cipher |
| Bypassing access control via email parsing discrepancies | مكتبة validation ومكتبة إرسال بيفسروا UTF-7 مختلف | استخدم نفس منطق parsing في كل مكان يتعامل مع نفس البيانات |

## 7️⃣ Web Cache Poisoning / Host Header / SSRF

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| Cache poisoning via ambiguous requests | Host header مكرر: back-end وcache بيفسروه مختلف | أي هيدر مؤثر على المحتوى لازم يدخل في الـ cache key |
| Routing-based SSRF | Host header بيتحكم في الـ internal routing | لا تثق بـ Host header لاتخاذ قرارات routing داخلية |
| SSRF via flawed request parsing | Absolute URL في request line بديل عن Host header | افحص كل مصادر الـ host بنفس الصرامة |
| Host validation bypass via connection state | فحص بيحصل مرة واحدة بس لكل TCP connection | كل طلب لازم يتفحص لوحده، مش يرث ثقة من طلب سابق |
| Password reset poisoning via dangling markup | Host port بيتحقن في رابط + raw HTML مش sanitized | Dangling markup بديل XSS لما الـ sanitization قوي |

## 8️⃣ OAuth / OpenID

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| Auth bypass via OAuth implicit flow | Client بيثق بإيميل جاي كـ parameter بدل استخراجه من الـ token | استخرج بيانات الهوية من الـ OAuth service مباشرة (server-to-server) |
| SSRF via OpenID dynamic client registration | `logo_uri` بيولّد طلب HTTP من السيرفر | افحص أي URL يُقبل من عميل قبل ما السيرفر "يزوره" |
| Forced OAuth profile linking | غياب `state` parameter = CSRF على عملية الربط | `state` أساسي لمنع CSRF في OAuth |
| OAuth account hijacking via redirect_uri | `redirect_uri` بدون whitelist صارمة | تحقق تام (exact match) من `redirect_uri` |
| Stealing tokens via open redirect | Path traversal في redirect_uri + open redirect داخلي | تسلسل ثغرات: traversal + open redirect + implicit flow ضعيف |
| Stealing tokens via a proxy page | `postMessage(*)` في iframe الكومنتات | حدد target origin صريح دايمًا في postMessage |

## 9️⃣ File Upload

| اللاب | السبب الجذري | الدرس الأساسي |
| --- | --- | --- |
| Web shell upload via extension blacklist bypass | Blacklist + رفع `.htaccess` لتعريف امتداد جديد | استخدم Whitelist بدل Blacklist |
| Web shell upload via obfuscated extension | Null byte: `exploit.php%00.jpg` | نظّف الاسم بالكامل + استخدم أسماء عشوائية من السيرفر |
| Web shell upload via race condition | الملف بيتحرك قبل التحقق منه، وبيتشال لو فشل | تحقق أولاً في مكان غير قابل للتنفيذ، انقل بعدين فقط |

## 🎯 المبادئ العامة المتكررة عبر كل الفئات

| المبدأ | أمثلة تطبيقه |
| --- | --- |
| **لا تثق بمدخلات العميل لاتخاذ قرارات أمنية** | Host header, hidden username fields, redirect_uri, client-supplied email |
| **الفحص الجزئي دايمًا قابل للتخطي** | Blacklists, `../` stripping, startswith() checks |
| **التناقض بين طبقتين = ثغرة كامنة** | Parser discrepancies (email, Host, cache vs backend) |
| **رسائل الخطأ لازم تكون موحدة** | Username enumeration, password change oracle |
| **كل خطوة في عملية متعددة الخطوات تحتاج فحصها الخاص** | Multi-step access control, 2FA logic, OAuth linking |
| **الأسرار المكشوفة مرة = محروقة للأبد** | Git history, hardcoded secrets |
| **دمج ثغرات صغيرة = هجوم كبير** | Path traversal + Host header, Open redirect + OAuth |
