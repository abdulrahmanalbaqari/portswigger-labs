# Lab: Basic SSRF against another back-end system

https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system

- الفكره من الLab ده ان parameter مصاب بيخليني اعرف اتعامل مع internal server بس المراضي انا مش بستهدف نفس الserver لا انا بستهدف server تاني علش نفس الشبكه فا المفروض اجرب الip rang 1:255

```bash
1. الparameter المصاب هو Check stock جواه الparameter المصاب stockApi 

2. stockApi=http%3A%2F%2F192.168.0.14%3A8080%2Fadmin ==>  admin panal  الي هيودي ني علي  IPبعد التجربه ده هوال

3. stockApi=http%3A%2F%2F192.168.0.14%3A8080%2Fadmin/delete?username=carlos ==> الحل او الاستغلال
```

نفس آلية "Check stock": الـ front-end بيبعت request فيه parameter اسمه `stockApi` بيحدد الـ URL اللي السيرفر هيعمله request عليه عشان يجيب بيانات المخزون. بس هنا الـ URL بيشاور على IP خاص (زي `192.168.0.X`) بدل localhost، وده معناه إن فيه سيرفر تاني جوه الشبكة الداخلية (internal network) مش متاح للعامة، لكن السيرفر الأساسي عنده وصول ليه.

بما إن الـ app مش بتعمل أي تحقق حقيقي على الـ host اللي بتبعتله، ممكن نستبدل الـ IP والـ port بتوع سيرفر التخزين بـ IP وport لسيرفر داخلي تاني (زي الـ admin interface)، والسيرفر هيوصله بالنيابة عننا.

#### إزاي بتتحل

1. افتح صفحة منتج ودوس "Check stock" وأنت شغّال Burp Proxy.
2. هتلاقي الـ request فيه حاجة زي:

```
stockApi=http://192.168.0.12:8080/product/stock/check?productId=1&storeId=1
```

1. الـ IP بتاع الباك-إند بيبقى غالبًا معروف من إن نفس الـ subnet مستخدم للسيرفر الأساسي، فالمطلوب إنك **تعمل Intruder attack** (أو تجرب يدوي) على آخر octet بتاع الـ IP، بحيث تجرب أرقام من 1 لـ 255 مع ports شائعة (زي 8080)، وده باستخدام رينج زي:

```
stockApi=http://192.168.0.1-255:8080/admin
```

1. راقب الـ responses: أي IP بيرجعله admin panel بدل الـ stock error هو الـ back-end system المستهدف.
2. لما تلاقيه، استخدم نفس الطريقة اللي فاتت — روح لـ `/admin` على الـ IP ده وشوف اللينكات زي `/admin/delete?username=carlos`، وابعتها كـ `stockApi` عشان تنفذ الأكشن وتكمل الـ lab.

#### فايدته إيه

- بيوضح إزاي SSRF ممكن تُستخدم كـ **"port scanner" واكتشاف سيرفرات داخلية** الشبكة الداخلية بالكامل مش بس السيرفر نفسه — يعني الـ blast radius أكبر بكتير من اللي فات.
- بيوضح خطورة الاعتماد على "الشبكة الداخلية معزولة فمفيش داعي لتحقق إضافي" — لأن أي سيرفر عنده SSRF بيبقى بوابة (pivot point) لكل حاجة تانية موصولة بيه في الشبكة.
- بيدرب على استخدام Burp Intruder لعمل enumeration للـ IP ranges، وهي مهارة مهمة في اختبار الاختراق (pentesting) للبنية الداخلية.
