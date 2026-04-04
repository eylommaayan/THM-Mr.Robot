# THM-Mr.Robot



שלב 1: איסוף מידע וסריקת רשת (Reconnaissance)
1.1 סריקת פורטים (Nmap)
השלב הראשון בתהליך היה זיהוי שירותים פעילים על גבי שרת היעד. ביצעתי סריקת nmap הכוללת זיהוי גרסאות שירותים (-sV) והרצת סקריפטים ברירת מחדל (-sC) כדי למפות את שטח התקיפה.

Bash
# הרצת סריקת nmap ראשונית
nmap -sV -sC -oA scans/initial 10.114.168.48

<img width="921" height="463" alt="image" src="https://github.com/user-attachments/assets/7f587d16-1d83-4670-9b10-36d51400d1cf" />

ממצאי הסריקה:
הסריקה העלתה שלושה פורטים פתוחים:

Port 22 (SSH): פתוח (גרסה OpenSSH 8.2p1).

Port 80 (HTTP): פתוח (שרת Apache).

Port 443 (HTTPS): פתוח (שרת Apache עם תמיכת SSL).

1.2 בחינת ממשק האינטרנט
בגישה לכתובת ה-IP של היעד דרך הדפדפן (http://10.114.168.48), מוצג טרמינל אינטראקטיבי המעוצב ברוח הסדרה "Mr. Robot". למרות שהממשק מרשים ויזואלית, הוא אינו מכיל פגיעות ישירה בשכבה הגלויה.

המשך התהליך יתמקד בביצוע Enumeration (מנייה) לספריות וקבצים חבויים בשרת כדי למצוא נקודת כניסה למערכת.

שלב 2: מעבר מסריקת רשת לסריקת אפליקציה (Web Enumeration)
הלוגיקה מאחורי השלב:
לאחר שסריקת ה-nmap זיהתה שפורט 80 (HTTP) פתוח, נקודת ההנחה היא שקיים שרת אינטרנט פעיל המשמש כווקטור תקיפה פוטנציאלי. בשלב זה, המטרה היא "לחשוף" קבצים או ספריות שאינם גלויים לעין בתפריטי האתר הרגילים.

זיהוי קבצי הגדרות (robots.txt):
כחלק מנוהל עבודה סטנדרטי (Best Practice), בדקתי את קובץ ה-robots.txt. קובץ זה נועד להנחות סורקי אינטרנט (כמו Googlebot) לאילו נתיבים לא להיכנס, ולכן הוא מהווה מקור מידע רגיש עבור האקרים, שכן הוא חושף לעיתים קרובות נתיבי ניהול או קבצי גיבוי.

ביצוע הפעולה:
ניגשתי לכתובת: http://10.114.168.48/robots.txt

<img width="1102" height="500" alt="image" src="https://github.com/user-attachments/assets/27359a4f-f02f-421c-a58a-4c74d7533965" />




2.3 השגת הדגל הראשון (Key 1 of 3)
לאחר שזיהיתי את הנתיב בקובץ ה-robots.txt, ניגשתי ישירות לקובץ הטקסט בדפדפן.

כתובת המקור:
http://10.114.168.48/key-1-of-3.txt

התוצאה:
הקובץ הכיל את המחרוזת הבאה (הדגל הראשון):
073403c9325489496e282b130f14652c
<img width="895" height="298" alt="image" src="https://github.com/user-attachments/assets/ef9bb2ba-e2f8-494f-8dc7-c426adef736d" />

<img width="1505" height="582" alt="image" src="https://github.com/user-attachments/assets/79e89750-9568-4173-b60f-ae6bf0b6199e"
  
  
  
  
  />


שלב 3: הורדת מילון המילים(fsocity.dic)
לאחר שזיהיתי את קובץ המילון ב-robots.txt, ניגשתי אליו כדי להוריד אותו. קובץ זה מכיל רשימת מילים המותאמת אישית לשרת זה, והוא ישמש אותי בשלבים הבאים לביצוע מתקפות Brute Force.

פעולה:
גישה לכתובת: http://10.114.174.154/fsocity.dic



<img width="1219" height="899" alt="image" src="https://github.com/user-attachments/assets/5152522f-7880-4ca6-9df0-f05f91b48bbe" />





<img width="599" height="498" alt="image" src="https://github.com/user-attachments/assets/a5a27ab8-044c-449c-9e2f-0e11cf51bca0" />

שלב 4: אופטימיזציה של רשימת המילים (Deduplication)
קובץ המילון שנמצא (fsocity.dic) הכיל כמות מאסיבית של מילים כפולות שהיו מעכבות את ניסיון הפריצה. ביצעתי תהליך סינון כדי להשאיר רק מילים ייחודיות.

הפקודות לביצוע:

Bash
sort fsocity.dic | uniq > fs-list.txt
אימות התוצאה:
באמצעות הפקודה wc -w, וידאתי שרשימת המילים צומצמה למספר יעיל של מילים ייחודיות.

Bash
wc -w fs-list.txt
# Output: 11451
<img width="1121" height="224" alt="image" src="https://github.com/user-attachments/assets/7491244d-bf7d-4e54-82d6-6e6350db4c42" />



שלב 5: סריקת ספריות (Directory Brute-Forcing)
כדי לחשוף נתיבים חבויים בשרת האינטרנט, השתמשתי בכלי gobuster. הסריקה נועדה למצוא דפי ניהול או ממשקי התחברות.

פקודה:

Bash
gobuster dir -u http://10.114.174.154/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
ממצאים בולטים:
הסריקה העלתה מספר נתיבים קריטיים המצביעים על כך שהאתר מריץ WordPress:

/wp-login.php (דף התחברות)

/wp-admin/ (פאנל ניהול)

/license





/readme
<img width="1587" height="834" alt="image" src="https://github.com/user-attachments/assets/c11c7cd5-ce27-46e1-811f-e9d78c3325cc" />


שלב 6: פריצה למערכת הניהול (Authentication Brute Force)
לאחר שזיהיתי שהאתר מריץ WordPress והנתיב /wp-login.php נגיש, עברתי לניסיון השגת גישה באמצעות הכלי Hydra. התקיפה התחלקה לשני שלבים:

8.1 זיהוי שם משתמש (Username Enumeration)
מכיוון ש-WordPress מחזירה הודעות שגיאה שונות עבור שם משתמש שגוי לעומת סיסמה שגויה, ניתן להשתמש בזה כדי למצוא שם משתמש תקף. השתמשתי במילון המילים שנוקה קודם לכן (fs-list) כדי לסרוק שמות משתמש פוטנציאליים.

הפקודה לזיהוי המשתמש:

Bash
hydra -L fs-list -p test 10.112.177.29 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username" -t 30

ממצא: שם המשתמש שזוהה במערכת הוא elliot.


<img width="1693" height="225" alt="image" src="https://github.com/user-attachments/assets/44f03dc9-bd43-4f65-900a-093799a995dd" />
8.2 פיצוח הסיסמה (Password Brute Force)
לאחר אימות שם המשתמש, הרצתי מתקפת Brute Force ממוקדת על הסיסמה של המשתמש elliot באמצעות המילון המזוקק.





הפקודה לפיצוח הסיסמה:

Bash
hydra -l elliot -P fs-list 10.112.177.29 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered" -t 64

תוצאה: הסיסמה נמצאה בהצלחה ER28-0652




<img width="1665" height="190" alt="image" src="https://github.com/user-attachments/assets/ca5c9d76-33c8-4cab-8184-f7b627d7016d" />




<img width="1632" height="879" alt="image" src="https://github.com/user-attachments/assets/ebceb381-6d39-486a-ac47-18fcd4e9ad34" />




<img width="1647" height="876" alt="image" src="https://github.com/user-attachments/assets/fb6dda89-16a4-473f-a1cd-7eafaed85694" />



שלב 10: כניסה לממשק הניהול והכנת ה-Reverse Shell
לאחר פיצוח הסיסמה בהצלחה, התחברתי למערכת הניהול של האתר. הגישה לפאנל הניהול מאפשרת למנהל המערכת לערוך קבצי קוד ישירות מהדפדפן, תכונה אותה ננצל להרצת קוד עוין.

1. אימות גישה ל-Dashboard:
כפי שניתן לראות בצילום המסך, התחברתי בהצלחה כמשתמש elliot. המערכת מזהה גרסת WordPress ישנה (4.3.1), מה שמעיד על פוטנציאל גבוה לחולשות נוספות.

<img width="749" height="477" alt="image" src="https://github.com/user-attachments/assets/d0cd68fa-a34f-4cb5-873d-d3a126d704e6" />



2. עריכת תבנית האתר (Theme Editor):
ניווטתי לתפריט Appearance ולאחר מכן ל-Editor. בחרתי לערוך את תבנית ה-404 Template (404.php) של ערכת הנושא "Twenty Fifteen".

הלוגיקה:
דף ה-404 הוא קובץ PHP שנטען בכל פעם שמשתמש ניגש לנתיב שלא קיים בשרת. על ידי החלפת הקוד המקורי ב-PHP Reverse Shell, אוכל להפעיל את הקוד שלי פשוט על ידי גלישה לכתובת אקראית באתר.

<img width="1716" height="841" alt="image" src="https://github.com/user-attachments/assets/279379b3-1310-45a0-b335-e171a98fe2cf" />








שלב 11: יצירת ה-Payload באמצעות Reverse Shell Generator
כדי להבטיח שהקוד יפעל בצורה תקינה ויחזור לכתובת הנכונה, השתמשתי באתר revshells.com. כלי זה מאפשר יצירת סקריפטים מוכנים מראש (Payloads) עבור מגוון רחב של שפות תכנות ומערכות הפעלה.

הגדרות שנבחרו (כפי שמוצג בצילום המסך):


<img width="1581" height="896" alt="image" src="https://github.com/user-attachments/assets/9c5408b4-9311-410a-9fa6-a5ee1e0c23c5" />


ביצוע:
העתקתי את הקוד שנוצר בחלון ה-Preview  כדי להדביק אותו בתוך עורך התבניות -WordPress.






שלב 12: פתרון בעיות והשגת גישה ראשונית (Initial Access)
במהלך הניסיונות להריץ את ה-Reverse Shell, נתקלתי במספר מכשולים טכניים שדרשו אבחון וטיפול לפני קבלת הגישה לשרת.

1. האתגרים והפתרונות (Troubleshooting):
בעיית נתיב הקובץ: בתחילה, הקוד הוזרק לקובץ העיצוב (style.css), שאינו מורץ על ידי השרת. הפתרון היה זיהוי התבנית הפעילה (Twenty Fifteen) והזרקת הקוד לקובץ ה-404.php (Template), שמאפשר הרצת קוד PHP בצד השרת.

שגיאת הגדרת IP (LHOST): הקוד המקורי הכיל את כתובת ה-IP של היעד במקום את כתובת המכונה התוקפת. עדכנתי את ה-Payload לכתובת ה-VPN המדויקת שלי (192.168.203.195).

חסימת פורטים: הפורט הראשוני (1234) לא הגיב, לכן העברתי את ה-Payload ואת המאזין (Listener) לפורט 4444 המקובל יותר.

2. הפריצה וקבלת ה-Shell:
לאחר תיקון הפרמטרים, הרצתי מאזין באמצעות Netcat וניגשתי בדפדפן לנתיב הישיר של הקובץ:
http://10.112.177.29/wp-content/themes/twentyfifteen/404.php

<img width="1055" height="547" alt="image" src="https://github.com/user-attachments/assets/6ef0aa95-8ac0-4f60-9b5a-1d52bb421887" />



<img width="1198" height="231" alt="image" src="https://github.com/user-attachments/assets/ac67327b-2c12-4966-8e67-264ab2a34bb6" />


התוצאה:
החיבור בוצע בהצלחה וקיבלתי גישת טרמינל (Interactive Shell) כמשתמש המערכת daemon.



שלב 15: שדרוג ה-Shell (Interactive Shell Upgrade)
לאחר קבלת הגישה הראשונית, ה-Shell שהתקבל היה מוגבל מאוד (Non-Interactive). כדי לאפשר עבודה נוחה יותר, הרצת פקודות מורכבות ושימוש בקיצורי מקלדת, ביצעתי שדרוג ל-TTY Shell באמצעות Python.

הפעולה שבוצעה:
השתמשתי במודול ה-pty של Python כדי להריץ תהליך Bash חדש בסביבת הטרמינל הקיימת.

הפקודה להרצה:

Bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
התוצאה:
הטרמינל השתנה מ-$ פשוט לשורה המציגה את שם המשתמש והמכונה (daemon@ip-10-112-177-29:/$). פעולה זו מאפשרת לי כעת לנווט במערכת הקבצים בצורה יעילה הרבה יותר.



<img width="1089" height="214" alt="image" src="https://github.com/user-attachments/assets/c9896f18-42e7-4b26-8b43-efde7a329992" />




