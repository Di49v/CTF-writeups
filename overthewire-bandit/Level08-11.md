# Bandit Wargame: Levels 7 to 11

**Date:** 2025-12-04
**Difficulty**: Easy
**Key Concepts:** Line-wise operations on files.

### Level 7 -> 8 
* **Objective:** The password for the next level is stored in the file **data.txt** next to the word **"millionth"**
* **Solution:**
    ```bash
    grep "millionth" data.txt
    ```
* **Key Learning:** Learnt the `grep` command and also tried `-A1` `-B1` options, for the sake of curiosity. So, actually I used this:
```bash
grep -A1 -B1 "millionth" data.txt
```

---
### Level 8 -> 9
* **Objective:** The password for the next level is stored in the file **data.txt** and is the only line of text that occurs only once.
* **Solution:**
    ```bash
    sort data.txt | uniq -u
    ```
    OR
    ```bash
    sort < data.txt | uniq -u
    ```
* **Key Learning:** Got confused first as when I saw the manual for `uniq`, it didn't mention at the start that it works only on adjacent lines, so as I was not using `sort` earlier so it returned a lot of lines, but then I googled for `uniq` command and the first line I got was that it works on adjacent lines, so I sorted it out and it worked out.

---
### Level 9 -> 10
* **Objective:** The password for the next level is stored in the file **data.txt** in one of the few human-readable strings, preceded by several ‘=’ characters.
* **Solution:**
    ```bash
    strings data.txt | grep "===="
    ```
    *It returned the password is [password]*
* **Key Learning:** Learnt `strings` .

---

### Level 10 -> 11
* **Objective:** The password for the next level is stored in the file **data.txt**, which contains base64 encoded data.
* **Solution:**
    ```bash
    base64 < data.txt
    ```
* **Key Learning:** Learnt `base64` .

---
