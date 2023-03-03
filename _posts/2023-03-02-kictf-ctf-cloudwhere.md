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

![image-20230301054111532](/assets/img/ctf/kitctf/cloudwhere/cat_flag.png)

נעבור לקוד הראשי של האפליקציה. ב- server.ts ניתן לראות את כל ה- routes של האתר:

![image-20230301054228233](/assets/img/ctf/kitctf/cloudwhere/routes.png)

נחזור ל-routes. אחרי שעברתי על כל אחד מהם, מצאתי את ה- route הדיפולטי כמעניין - index:

![image-20230301054424175](/assets/img/ctf/kitctf/cloudwhere/index_function.png)


הוא מחליף את ה- FOOTER בדף בpayload אחר ונעזר ב- ipToCountryCode:

![image-20230301054437824](/assets/img/ctf/kitctf/cloudwhere/ipToCountry_code.png)

![image-20230301054453599](/assets/img/ctf/kitctf/cloudwhere/ipToCountry_result.png)

הדבר הראשון שעולה לנו לראש זה: "איך אני עושה command injection לפרמטר ip המזורגג הזה. עכשיו מתחילים 😊
כדי לברר את זה הלכתי לכל הפניות לפונקציה הזאת וראיתי איך מגיעים הפרמטרים. ראיתי כי request.app.realIp מגיע מה- middleware.ts:

![image-20230301054504904](/assets/img/ctf/kitctf/cloudwhere/checkIpHeader.png)

מגניב ! בוא נשלח לשרת בקשה עם ה- Header: cf-connecting-ip ו"נזריק" לו את מה שאנחנו רוצים כדי שנשפיע על ה- whois command:

![image-20230301054538249](/assets/img/ctf/kitctf/cloudwhere/cf_header.png)

את הפלי ... מה עושים עכשיו ? למה זה קורה ?
Cf-connecting-ip בא להגיד לשרת מה ה- IP של הלקוח שפונה אליו.
השרת שאנחנו פונים אליו הינו מאחורי Cloudflare:
(תשובה עבור בקשת GET רגילה)

![image-20230301054608331](/assets/img/ctf/kitctf/cloudwhere/cf_response.png)

לכן כשאנחנו מוסיפים את ה- Header הזה אנחנו חוטפים שגיאה כי ככל הנראה השירות עצמו מעבד את הבקשה שהוא רואה header שמתחיל ב- cf. כדי לעקוף את זה ועדיין להגיע לפרמטר request.app.realIp הלכתי בדרך הבאה.

אחד מה- route'ים הינו proxy שמפנה עם הפרמטר endpoint לפונקציה הנ"ל:

![image-20230301054645581](/assets/img/ctf/kitctf/cloudwhere/proxy_request.png)

די אינטואיטיבי - הוא לוקח את הפרמטר endpoint כ- base64, מבצע פנייה בשמו לURL מסוים ומחזיר תשובה.
(מפתה לעשות SSRF אבל גם זה לא משרת את המטרה - לקרוא את הflag מהFS וגם לא עבד ... (:)

העליתי שרת ngrok כדי לראות איך הוא מבצע בקשות ושמתי לב שהוא ניגש מכתובת IP שונה:

![image-20230301054656307](/assets/img/ctf/kitctf/cloudwhere/ngrok_console.png)

![image-20230301054733716](/assets/img/ctf/kitctf/cloudwhere/ngrok_console_response.png)

כשאני ניגש לכתובת הזאת נראה שזה העתק של אותו האתר:

![image-20230301054747391](/assets/img/ctf/kitctf/cloudwhere/site_by_ip.png)

רק מה הפעם ...? השרת לא מאחורי Cloudflare !
עכשיו ננסה לפנות עם ה- cf-connecting-ip header:

![image-20230301054804018](/assets/img/ctf/kitctf/cloudwhere/burp_01.png)

🐱
אם תהיתם למה הוא מביא רק חצי מהתשובה תזכרו שהוא עושה split לפני שהוא מחזיר את התשובה (ראו את הקוד של הפונקציה ipToCountryCode). כעת בוא נתחכם:

![image-20230301054816859](/assets/img/ctf/kitctf/cloudwhere/base_requests.png)

מה שעשיתי כאן כדי להימנע מה- split זה שימוש ב- base64. פשוט אתרגם אותו בבית:

![image-20230301054834575](/assets/img/ctf/kitctf/cloudwhere/burp_02.png)

![image-20230301054848685](/assets/img/ctf/kitctf/cloudwhere/base64_output.png)

פשוט נשנה את הpayload לcat לקובץ:

![image-20230301054907285](/assets/img/ctf/kitctf/cloudwhere/flag_output.png)

😁
</div>
