---
layout: post
title: Winlogon
subtitle: Explanation of Kerberos protocol and the related attacks
categories: Windows Protocols
cover-img: /assets/img/kerberos/cerberus_dogs_cover_img.png
thumbnail-img: /assets/img/kerberos/cerberus_dogs_wallpaper.png
tags: [Mimikatz, LSASS, Winlogon]
comments: true
---
<div dir="rtl">
# איפה להתחיל ?

נתחיל מהסוף להתחלה, איפה הדגל? אפשר לראות ב- Dockerfile  את הניתוב של הדגל שאנחנו צריכים:

![image-20230301054111532](/home/nave/.config/Typora/typora-user-images/image-20230301054111532.png)

נעבור לקוד הראשי של האפליקציה. ב- server.ts ניתן לראות את כל ה- routes של האתר:

![image-20230301054228233](/home/nave/.config/Typora/typora-user-images/image-20230301054228233.png)

נחזור ל-routes. אחרי שעברתי על כל אחד מהם, מצאתי את ה- route הדיפולטי כמעניין - index:

![image-20230301054424175](/home/nave/.config/Typora/typora-user-images/image-20230301054424175.png)


הוא מחליף את ה- FOOTER בדף בpayload אחר ונעזר ב- ipToCountryCode:

![image-20230301054437824](/home/nave/.config/Typora/typora-user-images/image-20230301054437824.png)

![image-20230301054453599](/home/nave/.config/Typora/typora-user-images/image-20230301054453599.png)

הדבר הראשון שעולה לנו לראש זה: "איך אני עושה command injection לפרמטר ip המזורגג הזה. עכשיו מתחילים 😊
כדי לברר את זה הלכתי לכל הפניות לפונקציה הזאת וראיתי איך מגיעים הפרמטרים. ראיתי כי request.app.realIp מגיע מה- middleware.ts:

![image-20230301054504904](/home/nave/.config/Typora/typora-user-images/image-20230301054504904.png)

מגניב ! בוא נשלח לשרת בקשה עם ה- Header: cf-connecting-ip ו"נזריק" לו את מה שאנחנו רוצים כדי שנשפיע על ה- whois command:

![image-20230301054538249](/home/nave/.config/Typora/typora-user-images/image-20230301054538249.png)

את הפלי ... מה עושים עכשיו ? למה זה קורה ?
Cf-connecting-ip בא להגיד לשרת מה ה- IP של הלקוח שפונה אליו.
השרת שאנחנו פונים אליו הינו מאחורי Cloudflare:
(תשובה עבור בקשת GET רגילה)

![image-20230301054608331](/home/nave/.config/Typora/typora-user-images/image-20230301054608331.png)

לכן כשאנחנו מוסיפים את ה- Header הזה אנחנו חוטפים שגיאה כי ככל הנראה השירות עצמו מעבד את הבקשה שהוא רואה header שמתחיל ב- cf. כדי לעקוף את זה ועדיין להגיע לפרמטר request.app.realIp הלכתי בדרך הבאה.

אחד מה- route'ים הינו proxy שמפנה עם הפרמטר endpoint לפונקציה הנ"ל:

![image-20230301054645581](/home/nave/.config/Typora/typora-user-images/image-20230301054645581.png)

די אינטואיטיבי - הוא לוקח את הפרמטר endpoint כ- base64, מבצע פנייה בשמו לURL מסוים ומחזיר תשובה.
(מפתה לעשות SSRF אבל גם זה לא משרת את המטרה - לקרוא את הflag מהFS וגם לא עבד ... (:)

העליתי שרת ngrok כדי לראות איך הוא מבצע בקשות ושמתי לב שהוא ניגש מכתובת IP שונה:

![image-20230301054656307](/home/nave/.config/Typora/typora-user-images/image-20230301054656307.png)

![image-20230301054733716](/home/nave/.config/Typora/typora-user-images/image-20230301054733716.png)

כשאני ניגש לכתובת הזאת נראה שזה העתק של אותו האתר:

![image-20230301054747391](/home/nave/.config/Typora/typora-user-images/image-20230301054747391.png)

רק מה הפעם ...? השרת לא מאחורי Cloudflare !
עכשיו ננסה לפנות עם ה- cf-connecting-ip header:

![image-20230301054804018](/home/nave/.config/Typora/typora-user-images/image-20230301054804018.png)

🐱
אם תהיתם למה הוא מביא רק חצי מהתשובה תזכרו שהוא עושה split לפני שהוא מחזיר את התשובה (ראו את הקוד של הפונקציה ipToCountryCode). כעת בוא נתחכם:

![image-20230301054816859](/home/nave/.config/Typora/typora-user-images/image-20230301054816859.png)

מה שעשיתי כאן כדי להימנע מה- split זה שימוש ב- base64. פשוט אתרגם אותו בבית:

![image-20230301054834575](/home/nave/.config/Typora/typora-user-images/image-20230301054834575.png)

![image-20230301054848685](/home/nave/.config/Typora/typora-user-images/image-20230301054848685.png)

פשוט נשנה את הpayload לcat לקובץ:

![image-20230301054907285](/home/nave/.config/Typora/typora-user-images/image-20230301054907285.png)

😁
</div>
