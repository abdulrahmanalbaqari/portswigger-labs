# Lab: Blind SSRF with out-of-band detection

https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection

- الفكره من الLab ده ان الموقع مفيش فيه parameter تخليني اتعامل مع الinternal server بس الReferer header مش متامنه كويس هو كدا كدا بيمر عليها فا استخدمت ال burp collaborator  علشان احل الlab

```bash
1. Referer: https://g5m45nf6wz2mvafnjzp9nddybphg5ft4.oastify.com/
```

في الـ lab، لما تدخل صفحة منتج، السيرفر بيبعت طلب HTTP لخدمة تحليلات (analytics) — الطلب ده بيتحدد من خلال الـ **`Referer` header** اللي المتصفح بيبعته تلقائيًا. يعني السيرفر بياخد قيمة الـ Referer ويستخدمها كـ URL يعمله request عليه (زي: يبعت بيانات analytics لنفس الـ domain اللي جاي منه الـ referer).

المشكلة: مفيش أي تحقق من إن الـ Referer ده domain حقيقي أو تابع للموقع، فلو غيّرناه لـ domain احنا متحكمين فيه، السيرفر (من غير علمنا بأي response) هيبعتله request فعليًا.

#### إزاي بتتحل

1. سجّل حساب مجاني على **Burp Collaborator** (أو استخدم الـ Collaborator client المدمج في Burp Suite Professional).
2. افتح صفحة أي منتج وأنت شغّال Burp Proxy، والقط الـ request بتاعها.
3. هتلاقي فيه:

```
Referer: https://your-lab-id.web-security-academy.net/product?productId=1
```

1. غيّر قيمة الـ `Referer` لـ subdomain عشوائي من الـ Collaborator بتاعك:

```
Referer: http://your-collaborator-id.oastify.com
```

1. ابعت الـ request (أو خليه يتبعت طبيعي بالتصفح).
2. روح لـ **Burp Collaborator client** ودوس "Poll now" — لو ظهرلك DNS و/أو HTTP interaction جاي من السيرفر بتاع الـ lab، يبقى كده أثبتّ إن فيه SSRF (حتى من غير ما تشوف أي رد فعل في الـ response بتاعك).

كده الـ lab يعتبر solved لمجرد إثبات الـ out-of-band interaction.

#### فايدته إيه

- بيوضح إن مش كل SSRF بترجع لك response واضح — كتير من الحالات الواقعية بتكون **blind**، فلازم تعرف تكتشفها بطريقة تانية غير إنك تشوف نتيجة في المتصفح.
- بيعرّفك على **Burp Collaborator**، وهو أداة أساسية في اختبار أي ثغرة "blind" (زي blind SQLi, blind XXE, blind SSRF) عن طريق تسجيل تفاعلات DNS/HTTP خارجية.
- بيوضح إن أي header بيتبعت من المتصفح (زي `Referer`, `X-Forwarded-For`, `User-Agent`) ممكن يبقى مصدر خطر لو الـ back-end استخدمه في عمل request من غير تحقق — مش بس الـ parameters الظاهرة في الـ body.
- الخطوة دي عادة بتكون أول خطوة قبل استغلال blind SSRF فعليًا (زي إنك تحاول تستخدمه لعمل internal port scanning حتى من غير ما تشوف نتيجة مباشرة، بالاعتماد على الفرق في زمن الاستجابة أو عدد الـ interactions).
