# Lab : Basic server-side template injection

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic

- الفكره من ال Lab ده انك تستغل ان الموقع مصاب ب ثغره SSTI و تتحكم في الserver وتمسح ملف لشخص معين

```bash
1. <%= 7 * 7 %>

2. <%= File.delete("/path/to/file") %>

3. <%= system("rm /home/carlos/morale.txt") %>
```

بتحصل لما التطبيق يستخدم **template engine** (زي Jinja2, Twig, FreeMarker, ERB, Velocity...) عشان يولّد صفحات ديناميكية، وبيحط **مدخلات المستخدم مباشرة جوه الـ template نفسه** بدل ما يمررها كـ "بيانات" بس.

النتيجة: بدل ما الـ engine يعامل الكلام بتاعك كـ نص عادي، بيعامله كـ **كود/تعبير (expression)** لازم يتنفذ ويترجم. لو قدرت تحقن syntax بتاع الـ engine، ممكن توصل لـ **Remote Code Execution (RCE)** على السيرفر.

### اللاب "Basic server-side template injection"

اللاب ده بسيط ومباشر، وبيقدملك مثال أساسي لأول مرة تتعامل مع SSTI:

**السيناريو:** الموقع فيه فيتشر "Feedback" أو "Submit feedback"، وبعد ما تبعت الرسالة، الرد اللي بيظهرلك بيتعمله render باستخدام **template engine اسمه ERB** (بتاع Ruby)، وقيمة الرسالة بتاعتك بتتحط جوه الـ template من غير أي فلترة.

**خطوات الحل:**

1. تكتشف الثغرة عن طريق إنك تحط تعبير رياضي زي `${7*7}` — لو الناتج رجعلك `49` بدل ما يظهر النص زي ما هو، يبقى في injection حصل.
2. تتأكد إن الـ engine هو **ERB** (syntax بتاعه: `<%= expression %>`)
3. تستخدم صلاحية ERB في تنفيذ كود Ruby عشان تنفذ أمر على نظام التشغيل، غالبًا باستخدام:

```
   <%= system("rm /home/carlos/morale.txt") %>
```

1. الهدف النهائي: حذف ملف `morale.txt` من الـ home directory بتاع Carlos — ده دليل على إنك حققت RCE فعلي.

### الفايدة من اللاب

1. **تفهم الفرق الجوهري** بين إن الداتا بتتحط "كـ قيمة" جوه متغير، وبين إنها "تتحقن" جوه التمبلت نفسه فتتفسر كـ كود.
2. **بتاخد أول تجربة عملية** لـ SSTI من غير تعقيد إضافي (زي WAF أو sandboxing) — عشان تفهم الميكانيزم الأساسي الأول قبل ما تتعامل مع لابات أصعب زي:
    - Basic SSTI with information disclosure
    - SSTI in a sandboxed environment
    - SSTI using documentation (اللي شرحناه قبل كده)
3. **بيوريك إزاي RCE ممكن يوصل من مكان بسيط** زي فورم feedback عادي — يعني أي مكان بيتقبل مدخلات وبيتعمله render ممكن يبقى نقطة دخول خطيرة.
4. **بيأسس لفهم الحماية الصح**: الحل السليم إنك متسمحش أبدًا بدمج user input جوه الـ template syntax — استخدم الـ template فقط كـ "قالب ثابت" واحط قيم المستخدم كـ **بيانات (data)** بتتمرر له، مش كـ جزء من الكود نفسه.
