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




שלב 4: סריקת ספריות (Directory Brute-Forcing)
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


