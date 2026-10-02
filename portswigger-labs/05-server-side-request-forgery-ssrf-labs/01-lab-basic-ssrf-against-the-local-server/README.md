# Lab : Basic SSRF against the local server

https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost

- الفكره من ال Lab ده ان الموقع في parameter مصاب بSSRF بيخليني اتعامل مع الserver والمفروض استغل وروح لواجهت ال Admin ومسح في user

```bash
1. في parameter مصاب وهو Check stock والي بايظ فيه ان في stockApi= بياخد اي روابط وبينفزها فا هنجرب 

2. http://localhost/admin ==> نفع بس مشعارف اعدل في الصفحه واخد الصلاحيات بس اتاكت ان الجزء ده مصاب فا هنجرب الاساغلال

3. stockApi=http://localhost/admin/delete?username=carlos 
```

في الموقع بتاع الـ lab في صفحة منتج فيها زرار "Check stock". لما تدوس عليه، الـ front-end بيبعت request للـ server، والـ server (من جواه) بيعمل HTTP request لسيرفر تاني (سيرفر الـ stock) عشان يجيب حالة التوفر.

المشكلة إن الـ request ده بيتبعت بناءً على **URL بيجيله من الـ user نفسه** (غالبًا كـ parameter زي `stockApi`)، والـ server مش بيتحقق كويس من الـ URL ده قبل ما يعمله request. يعني لو غيّرت الـ URL، السيرفر (اللي عنده صلاحيات وصول للشبكة الداخلية) هو اللي هيبعت الـ request مش المتصفح بتاعك.

#### إزاي بتتحل

1. افتح صفحة أي منتج، ودوس "Check stock" وأنت شغّال الـ **Burp Suite Proxy/Intercept**.
2. هتلاقي request زي:

```
POST /product/stock HTTP/1.1
...
stockApi=http://stock.weliketoshop.net/product/stock/check?productId=1&storeId=1
```

1. غيّر قيمة الـ `stockApi` بحيث تخليه يشاور على الـ **localhost** بتاع السيرفر نفسه، وعلى بورت الـ admin interface (بتلاقيه غالبًا موصوف في الـ lab إنه على `/admin`):

```
stockApi=http://localhost/admin
```

1. ابعت الـ request، وهتلاقي في الـ response الرجوع لصفحة الـ admin panel (لأن الطلب اتبعت من جوه السيرفر نفسه، فهو اتعامل معاه كـ "trusted" internal request).
2. من صفحة الـ admin، هتلاقي روابط زي `/admin/delete?username=carlos` — استخدمها كـ `stockApi` تاني عشان تعمل الأكشن المطلوب (زي حذف يوزر معين اسمه carlos) وتكمل الـ lab.

#### فايدته إيه (ليه بيتعلموه)

- بيوضح إزاي ممكن الـ attacker يستغل السيرفر نفسه كـ **proxy** عشان يوصل لأماكن مش المفروض يوصلها (زي الشبكة الداخلية، admin panels، cloud metadata endpoints زي AWS)، من غير ما يكون عنده وصول مباشر ليها.
- بيعلّم أهمية إن أي URL جاي من الـ user لازم يتعمله **validation صارم** (whitelist للـ domains/IPs المسموحة) قبل ما السيرفر يستخدمه في request.
- من أخطر أنواع الثغرات لأنها بتكسر مبدأ "الشبكة الداخلية آمنة" اللي كتير من الأنظمة بتعتمد عليه.
