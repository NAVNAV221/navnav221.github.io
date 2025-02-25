---
layout: post
title: KITCTF - Cloudwhere
subtitle: Whitebox web ctf
categories: CTF Web
cover-img: /assets/img/ctf/kitctf/cloudwhere/cloudflare_logo.png
thumbnail-img: /assets/img/ctf/kitctf/kitctf_logo.png
tags: [NodeJs, Web, CTF, whitebox]
comments: true
---
<div dir="rtl">

# איפה להתחיל ?

</div>

<div dir="rtl">

נתחיל מהסוף להתחלה, איפה הדגל? אפשר לראות ב- Dockerfile  את הניתוב של הדגל שאנחנו צריכים:

</div>

![image-20230301054111532](/assets/img/ctf/kitctf/cloudwhere/cat_flag.png)

<div dir="rtl">

נעבור לקוד הראשי של האפליקציה. ב- server.ts ניתן לראות את כל ה- routes של האתר:

</div>

![image-20230301054228233](/assets/img/ctf/kitctf/cloudwhere/routes.png)

<div dir="rtl">

נחזור ל-routes. אחרי שעברתי על כל אחד מהם, מצאתי את ה- route הדיפולטי כמעניין - index:

</div>

![image-20230301054424175](/assets/img/ctf/kitctf/cloudwhere/index_function.png)

<div dir="rtl">

הוא מחליף את ה- FOOTER בדף בpayload אחר ונעזר ב- ipToCountryCode:

</div>

![image-20230301054437824](/assets/img/ctf/kitctf/cloudwhere/ipToCountry_code.png)

![image-20230301054453599](/assets/img/ctf/kitctf/cloudwhere/ipToCountry_result.png)

<div dir="rtl">

הדבר הראשון שעולה לנו לראש זה: "איך אני עושה command injection לפרמטר ip המזורגג הזה. עכשיו מתחילים 😊
כדי לברר את זה הלכתי לכל הפניות לפונקציה הזאת וראיתי איך מגיעים הפרמטרים. ראיתי כי request.app.realIp מגיע מה- middleware.ts:

</div>

![image-20230301054504904](/assets/img/ctf/kitctf/cloudwhere/checkIpHeader.png)

<div dir="rtl">

מגניב ! בוא נשלח לשרת בקשה עם ה- Header: cf-connecting-ip ו"נזריק" לו את מה שאנחנו רוצים כדי שנשפיע על ה- whois command:

</div>

![image-20230301054538249](/assets/img/ctf/kitctf/cloudwhere/cf_header.png)

<div dir="rtl">

את הפלי ... מה עושים עכשיו ? למה זה קורה ?
Cf-connecting-ip בא להגיד לשרת מה ה- IP של הלקוח שפונה אליו.
השרת שאנחנו פונים אליו הינו מאחורי Cloudflare:
(תשובה עבור בקשת GET רגילה)

</div>

![image-20230301054608331](/assets/img/ctf/kitctf/cloudwhere/cf_response.png)

<div dir="rtl">

לכן כשאנחנו מוסיפים את ה- Header הזה אנחנו חוטפים שגיאה כי ככל הנראה השירות עצמו מעבד את הבקשה שהוא רואה header שמתחיל ב- cf. כדי לעקוף את זה ועדיין להגיע לפרמטר request.app.realIp הלכתי בדרך הבאה.

אחד מה- route'ים הינו proxy שמפנה עם הפרמטר endpoint לפונקציה הנ"ל:

</div>

![image-20230301054645581](/assets/img/ctf/kitctf/cloudwhere/proxy_request.png)

<div dir="rtl">

די אינטואיטיבי - הוא לוקח את הפרמטר endpoint כ- base64, מבצע פנייה בשמו לURL מסוים ומחזיר תשובה.
(מפתה לעשות SSRF אבל גם זה לא משרת את המטרה - לקרוא את הflag מהFS וגם לא עבד ... (:)

העליתי שרת ngrok כדי לראות איך הוא מבצע בקשות ושמתי לב שהוא ניגש מכתובת IP שונה:

</div>

![image-20230301054656307](/assets/img/ctf/kitctf/cloudwhere/ngrok_console.png)

![image-20230301054733716](/assets/img/ctf/kitctf/cloudwhere/ngrok_console_response.png)

<div dir="rtl">

כשאני ניגש לכתובת הזאת נראה שזה העתק של אותו האתר:

</div>

![image-20230301054747391](/assets/img/ctf/kitctf/cloudwhere/site_by_ip.png)

<div dir="rtl">

רק מה הפעם ...? השרת לא מאחורי Cloudflare !
עכשיו ננסה לפנות עם ה- cf-connecting-ip header:

</div>

![image-20230301054804018](/assets/img/ctf/kitctf/cloudwhere/burp_01.png)

<div dir="rtl">

🐱
אם תהיתם למה הוא מביא רק חצי מהתשובה תזכרו שהוא עושה split לפני שהוא מחזיר את התשובה (ראו את הקוד של הפונקציה ipToCountryCode). כעת בוא נתחכם:

</div>

![image-20230301054816859](/assets/img/ctf/kitctf/cloudwhere/base_requests.png)

<div dir="rtl">

מה שעשיתי כאן כדי להימנע מה- split זה שימוש ב- base64. פשוט אתרגם אותו בבית:

</div>

![image-20230301054834575](/assets/img/ctf/kitctf/cloudwhere/burp_02.png)

![image-20230301054848685](/assets/img/ctf/kitctf/cloudwhere/base64_output.png)

<div dir="rtl">

פשוט נשנה את הpayload לcat לקובץ:

</div>

![image-20230301054907285](/assets/img/ctf/kitctf/cloudwhere/flag_output.png)

<div dir="rtl">

😁

</div>
