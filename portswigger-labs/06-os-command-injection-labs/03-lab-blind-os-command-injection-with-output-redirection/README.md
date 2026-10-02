# Lab : Blind OS command injection with output redirection

https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection

- الفكره من ال Lab ده انه blinded بس علشان احله لازم اعرض اعرف مكن تنفيز الامر واذاي اظهره هنفذه في function feedback وهعرضه من خانه الصوره علشان هيا بتحمل في الاول وتعمل reload وتاخد فتره

```bash
1. || whoami > /var/www/images/output.txt || ==> كود الاستغلال 

2. /image?filename=output.txt ==>الي بتعرض الصوره استغلتها علشان اعرض الاستغلال  function  دي 
```

في الـ Command Injection العادي (اللي مش blind)، لما تحقن أمر زي `; whoami` بتشوف نتيجته على طول في الـ response اللي راجع من السيرفر. لكن في الـ **Blind** version، السيرفر بينفذ الأمر بس **مش بيرجعلك أي output** في الـ response - يعني حتى لو نجحت تحقن الأمر، مش هتشوف نتيجته بالعين المجردة.

المشكلة اللي المعمل ده بيعلمها: إزاي تستغل الـ vulnerability وتقرأ نتيجة الأمر رغم إن السيرفر مش بيوريك أي حاجة؟

### الحل: Output Redirection

بما إن الـ output مش بيتعرض في الـ response، الحل إنك:

1. **تنفذ الأمر وتوجه (redirect) نتيجته لملف** جوه الـ web root، بحيث يبقى ممكن توصله بعدين عن طريق الـ browser مباشرة.

مثال على الـ payload:

```
example.com && whoami > /var/www/images/output.txt
```

هنا `>` بتاخد نتيجة `whoami` وتحطها جوه ملف اسمه `output.txt` في فولدر بيتقدم فيه static files (زي فولدر الصور).

1. **تفتح الملف من المتصفح مباشرة**، يعني تروح على:

```
https://target.com/image?filename=output.txt
```

وهنا هتلاقي نتيجة الأمر مكتوبة جوه الملف.

### خطوات الحل بالتفصيل (زي ما هي في PortSwigger)

1. تدور على مكان في التطبيق فيه functionality بتستخدم قيمة بتوصل لأمر نظام (زي "check stock" اللي بتاخد product ID و store location).
2. تجرب تحقن باستخدام `||`, `&&`, `;`, أو backtick علشان تتأكد إن فيه فعلاً command injection (ممكن تستخدم تقنية زي time delay: `& ping -c 10 127.0.0.1 &` وتشوف الـ response بياخد وقت أطول).
3. تحدد فولدر بيتقدم منه static content (زي `/var/www/static/` أو `/var/www/images/` - بيبقى معروف من هيكل الموقع، زي الصور اللي بتتعرض).
4. تحقن أمر بالشكل ده:

```
& whoami > /var/www/images/output.txt &
```

1. تعمل الـ request.
2. تفتح `/images/output.txt` في المتصفح وتشوف النتيجة (زي اسم اليوزر اللي شغال بيه السيرفر - وده بيبقى الـ solve condition للمعمل).

### ليه المعمل ده مهم (الفكرة التعليمية)

- بيوريك إن غياب الـ output مباشرة مش معناه إن الـ vulnerability مش موجودة أو مش قابلة للاستغلال (blind ≠ safe).
- بيعلمك إزاي تستغل معرفتك بهيكل السيرفر (فين الملفات الـ static) عشان تحول ثغرة "عمياء" لثغرة تقدر تقرأ نتيجتها.
- بيربط بين حاجتين: command injection + معرفة إزاي السيرفر بيقدم الملفات (static file serving) - وده مبدأ عام هتشوفه في vulnerabilities تانية كتير (زي file upload → RCE مثلاً).
