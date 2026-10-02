# Lab : SSRF with filter bypass via open redirection vulnerability

https://portswigger.net/web-security/ssrf/lab-ssrf-filter-bypass-via-open-redirection

- الفكره من ال Lab ده ان في parameter  مصاب بيخليني اتعامل مع الinternal server بس مسنفعش احط لنك خارجي مره واحده لازم redirect من الصفحه نفسها

```bash
1. stockApi=%2Fproduct%2FnextProduct%3fcurrentProductId%3d1%26path%3dhttp://192.168.0.12:8080/admin

2. stockApi=%2Fproduct%2FnextProduct%3fcurrentProductId%3d1%26path%3dhttp://192.168.0.12:8080/admin/delete?username=carlos
```

### الفكرة (SSRF via Open Redirect)

اللاب دي بتستخدم whitelist filter على الـ `stockApi` — يعني السيرفر بيتأكد إن الـ host اللي هيبعتله request يكون على نفس الدومين بتاع الموقع (أو subdomain معين زي `stock.weliketoshop.net`). فمينفعش تحط host تاني زي `http://169.254.169.254` أو أي internal IP مباشرة، هيترفض.

لكن فيه endpoint تاني (`/product/nextProduct?currentProductId=X&path=Y`) بيعمل **302 redirect** لأي `path` تديله — ده open redirect. والمهم: الـ endpoint ده نفسه على الـ host المسموح (الـ whitelist)، فالفلتر بيوافق عليه، لكن بعد ما السيرفر يعمل follow للـ redirect، بيوصل لأي مكان انت حاططه في `path` — حتى لو كان `localhost` أو IP داخلي.

### الخطوات العملية

1. في Burp Repeater، بدل الـ `stockApi` بقيمة زي دي (استبدل الدومين بدومين اللاب بتاعك):

```
stockApi=http://YOUR-LAB-ID.web-security-academy.net/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp://localhost/admin
```

لاحظ: لازم تعمل URL-encode لعلامات `?` و `&` اللي جوه الـ path عشان متتقطعش من الـ parameter الأساسي. يعني بدل `?` تحط `%3f` وبدل `&` تحط `%26`.

1. ابعت الـ request. لو نجح، هتلاقي في الـ response جوه الصفحة إن السيرفر عمل follow للـ redirect ووصل لـ `http://localhost/admin` وهيرجعلك محتوى صفحة الأدمن (زي قائمة يوزرز فيها زرار Delete جنب كل واحد).
2. من محتوى الصفحة دي هتلاقي رابط زي:

```
/admin/delete?username=carlos
```

1. كرر نفس الخطوة بس خلي الـ `path` يشاور على رابط الحذف ده بدل `/admin`:

```
stockApi=http://YOUR-LAB-ID.web-security-academy.net/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp://localhost/admin/delete%3fusername%3dcarlos
```

1. ابعتها → هيتنفذ الحذف → اللاب يتحل.

### ملحوظة مهمة

- جرب الأول تتأكد إن الـ `path` بيعمل فعلاً open redirect لوحده (يعني جرب `GET /product/nextProduct?currentProductId=1&path=https://example.com` وشوف هل بيرجع 302 لـ example.com ولا لأ).
- لو الـ whitelist عندك بيطلب host معين زي `stock.weliketoshop.net` مش نفس دومين اللاب، ابدأ الـ stockApi بيه بدل `YOUR-LAB-ID`.
- استخدم Burp Repeater مش المتصفح مباشرة، عشان تقدر تتحكم في الـ raw request وتشوف الـ response كامل.
