---
title: היכרות עם CICS
description: >
    פרק 2
weight: 10
---

ברוך הבא לעולם ה-**CICS (Customer Information Control System)**! בתור מתכנת COBOL טרי, סביר להניח שאתה כבר מכיר את היסודות של פיתוח יישומים בסביבת מיינפריים. CICS הוא רכיב קריטי נוסף בארכיטקטורת המיינפריים, המאפשר לאותם יישומי COBOL לתקשר עם משתמשי קצה ולספק שירותים אינטראקטיביים בזמן אמת.

מדריך זה יעזור לך להבין את עקרונות הליבה של CICS ויספק לך בסיס איתן להתחלת העבודה.

---

### מה זה CICS ולמה הוא חשוב?
CICS היא תוכנת ניהול טרנזקציות (**Transaction Manager**) מבית IBM, המאפשרת לאלפי משתמשים לגשת ולעבד נתונים בו-זמנית על מערכות מיינפריים. דמיין בנק שבו מאות פקידים ועוד אלפי לקוחות ניגשים לחשבונותיהם דרך אפליקציות שונות – CICS הוא המנוע שמאפשר את כל זה בצורה יעילה, מהירה ובטוחה.

**למה הוא חשוב למתכנת COBOL?**

* **אינטראקציה עם משתמשים:** בעוד COBOL מצוין בעיבוד נתונים בקבצים ובמסדי נתונים, CICS מספק את הממשק הדרוש לתקשורת עם מסופים (טרמינלים) ומאפשר למשתמשים להזין נתונים ולקבל תוצאות.
* **טרנזקציות:** CICS מנהל **טרנזקציות** – רצף של פעולות לוגיות המבוצעות כיחידה אחת (או שכולן מצליחות או שכולן נכשלות). זה חיוני לשמירה על שלמות הנתונים.
* **ביצועים ויעילות:** CICS מתוכנן לביצועים גבוהים ולניצול יעיל של משאבי המיינפריים, מה שמאפשר למערכות לטפל בעומסים כבדים.

---

### מושגי יסוד ב-CICS

כדי להתחיל לעבוד עם CICS, עליך להכיר כמה מונחים ומבנים בסיסיים:

* **טרנזקציה (Transaction):** יחידת עבודה לוגית ב-CICS. לכל טרנזקציה יש קוד ייחודי (בדרך כלל 4 תווים) המשמש להפעלה שלה. לדוגמה, טרנזקציה להצגת פרטי לקוח או לביצוע העברה בנקאית.
* **תוכנית (Program):** קוד COBOL (או שפות אחרות) המכיל את הלוגיקה העסקית. טרנזקציה מפעילה תוכנית אחת או יותר.
* **מפה (Map/Mapset):** רכיב חזותי המגדיר את הפריסה של מסך קלט/פלט. מפות משמשות להצגת נתונים למשתמש ולקבלת קלט ממנו. מפות נוצרות באמצעות כלי שנקרא **BMS (Basic Mapping Support)**.
כיום כבר לא משתמשים במפות.
* **מסוף (Terminal):** ההתקן הפיזי או הלוגי המאפשר למשתמש קצה לתקשר עם CICS (לרוב, מסוף 3270 או אמולטור שלו). כיום הנתונים מועברים החוצה אלא מערכות אחרות שממשיכות את שלב התצוגה ולא מטפלים ישירות בתצוגה למסוף.
* **משאבים Resource:** משאב במערכת, לדוג' קובץ, מסד נתונים, תור הודעות.


### מבנה תוכנית COBOL ב-CICS

תוכניות COBOL הפועלות תחת CICS דורשות כמה שינויים והוספות:

1.  **EXEC CICS Commands:**  
 אלו הן פקודות מיוחדות המשולבות בקוד COBOL ומאפשרות לתוכנית לתקשר עם CICS. הן מתורגמות על ידי Pre-compiler של CICS לקריאות לתוכניות שירות של CICS.
    * **דוגמאות לפקודות נפוצות:**
        * `EXEC CICS READ FILE`: קורא רשומה מקובץ VSAM.
        * `EXEC CICS WRITE FILE`: כותב רשומה לקובץ VSAM.
        * `EXEC CICS LINK PROGRAM`: מעביר שליטה לתוכנית אחרת.
        * `EXEC CICS RETURN`: מחזיר שליטה ל-CICS (מסמן סיום טרנזקציה או העברת שליטה חזרה לתוכנית קוראת).

2.  **DFHCOMMAREA (Communication Area):**
אזור זיכרון שמשמש להעברת נתונים בין תוכניות CICS שונות בתוך אותה טרנזקציה, או בין טרנזקציות (במקרים מסוימים). זהו מנגנון חשוב לשמירת "מצב" (state) בין מסכים שונים באותו תהליך עסקי.

    * **דוגמה לשימוש ב-DFHCOMMAREA:**
<div dir=ltr>

    ```cobol
    WORKING-STORAGE SECTION.
        01 WS-COMM-AREA.
            05 WS-CUSTOMER-ID    PIC X(10).
            05 WS-TRANSACTION-TYPE PIC X(01).
    LINKAGE SECTION.
        01 DFHCOMMAREA.
            05 LK-CUSTOMER-ID    PIC X(10).
            05 LK-TRANSACTION-TYPE PIC X(01).
    PROCEDURE DIVISION.
        IF EIBCALEN = ZERO
            MOVE 'INITIAL' TO WS-TRANSACTION-TYPE
        ELSE
            MOVE DFHCOMMAREA TO WS-COMM-AREA
        END-IF.
        * ... logic based on WS-TRANSACTION-TYPE ...
        MOVE WS-COMM-AREA TO DFHCOMMAREA.
        EXEC CICS RETURN
                   COMMAREA(DFHCOMMAREA)
                   LENGTH(LENGTH OF DFHCOMMAREA)
        END-EXEC.
    ```
</div>

3.  **EIBA (EXEC Interface Block Area):**
 אזור זיכרון ש-CICS ממלא במידע חשוב על הטרנזקציה הנוכחית, כגון קוד הטרנזקציה (**EIBCALEN**), מזהה המסוף (**EIBTRMID**), ושדות אחרים. אתה יכול לגשת לנתונים אלה בתוך תוכנית ה-COBOL שלך.

---

### תהליך פיתוח בסיסי ב-CICS

1.  **תכנון:** הגדרת דרישות, זרימת טרנזקציות ומבנה מסכים.
2.  **כתיבת קוד COBOL:** כתיבת לוגיקה עסקית, תוך שימוש בפקודות `EXEC CICS` וגישה לשדות המפות.
3.  **קומפילציה (Compilation):** תוכנית ה-COBOL עוברת קדם-קומפילציה על ידי CICS Pre-compiler (המטפל בפקודות `EXEC CICS`), ולאחר מכן קומפילציה רגילה.
4.  **קישור (Link-edit):** יצירת מודול הרצה (**Load Module**).
5.  **הגדרת משאבים ב-CICS:** יש להגדיר את הטרנזקציות, התוכניות, המפות והקבצים בתוך טבלאות המשאבים של CICS (לרוב באמצעות כלי ניהול כמו CEDA). הגדרה זו אומרת ל-CICS כיצד לטעון ולהריץ את הרכיבים השונים.
6.  **בדיקה (Testing):** הרצת הטרנזקציה ממסוף CICS ובדיקת תקינותה.

---

### טיפים למתחילים

* **התחל בקטן:** צור טרנזקציה פשוטה שמציגה מסך אחד, מקבלת קלט, ומציגה הודעה.
* **השתמש בדוגמאות:** חפש דוגמאות קוד CICS COBOL קיימות במערכת שלך או באינטרנט.
* **הכר את כלי הניפוי (Debuggers):** למד להשתמש בכלי ניפוי באגים של CICS (כמו Xpediter/CICS או InterTest) – הם יצילו לך שעות של עבודה.
* **הבן את זרימת הבקרה:** CICS הוא מונחה אירועים (event-driven). חשוב להבין איך CICS מעביר בקרה לתוכניות וכיצד התוכניות מחזירות בקרה ל-CICS.
* **ניהול שגיאות:** למד איך לטפל בשגיאות שמוחזרות על ידי פקודות `EXEC CICS` (לדוגמה, באמצעות `RESP` או `HANDLE CONDITION`).
* **תיעוד IBM:** תיעוד IBM ל-CICS הוא מקיף ביותר ויכול לספק תשובות כמעט לכל שאלה.

---

CICS הוא נושא רחב, ואל תתייאש אם הכל נראה מורכב בהתחלה. עם תרגול והתנסות, תרכוש את הידע הדרוש לך כדי לפתח יישומים חזקים ויעילים בסביבת CICS.

בהצלחה!

---