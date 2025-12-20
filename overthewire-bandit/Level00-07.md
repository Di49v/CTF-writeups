# Bandit Wargame: Levels 0 to 7

**Date:** 2025-12-01  
**Difficulty**: Very easy
**Concepts used:** [[nano]], [[IPQoS throughput]], [[ssh]], [[cat]], [[-exec]], [[find]], [[reset]]

---

## Level 0: Environment Setup & Troubleshooting
**Objective:** Connect to the Bandit server using SSH.

### Troubleshooting Network Issues
Before connecting, I encountered a "Broken Pipe" error related to the NAT adapter settings.
* **Fix:** Modified the client's SSH configuration to handle QoS throughput.
* **Command used:** `sudo nano /etc/ssh/ssh_config`
* **Configuration added:**
    ```text
    Host *
        IPQoS throughput
    ```
    *Saved using `Ctrl+O` and exited with `Ctrl+X`.*

### Connection
* **Command:** `ssh bandit0@bandit.labs.overthewire.org -p 2220`.
* **Credentials:** Username `bandit0` / Password `bandit0`.

---

## Level 1 : Basic File Handling

### Level 1 (The Readme)
* **Challenge:** Reading a password from a file named readme in home directory.
* **Solution:** 
    ```bash
    cat readme
    ```

---

## Levels 2 - 4: Advanced File Handling
**Key Concepts:** Relative paths, spaces in filenames, and hidden files.

### Level 2 (The Dash File)
* **Challenge:** Reading a password from a file named `-`.
* **Solution:** The `-` is interpreted as a standard input delimiter by commands like `cat`. To force it to be read as a file, I used a **relative path**.
    ```bash
    cat ./-
    ```
    *Note: I initially tried using just the delimiter `--`, but that did not work; the relative path method was successful.*

### Level 3 (Spaces in Filenames)
* **Challenge:** Reading a file whose name contains both dashes and spaces.
* **Solution:** I needed to escape the spaces to prevent the shell from treating them as separate arguments.
    ```bash
    cat ./--spaces\ in\ filename--
    ```

### Level 4 (Hidden Files)
* **Challenge:** Reading a password from a hidden file.
* **Solution:**
    1.  Used `ls -a` to list all files (including those starting with `.`).
    2.  Used relative path naming to read it.
    ```bash
    cat ./...Hiding-From-You
    ```

---

## Level 5 & 6: Searching by File Attributes
**Key Concepts:** `file` type identification and the `find` command.

### Level 5 (Human-Readable Files)
* **Objective:** The password is in the only human-readable file in the `inhere` directory.
* **Solution:**
    ```bash
    cat -file0*
    ```
	*This utilized wildcard expansion to read the content of every file. Then I estimated the file number (Guessed -file06 first but was -file07)*
* **Key Learning:** The `reset` command can be used to fix terminal formatting if a binary file messes up the display.

### Level 6 (Size Filters)
* **Objective:** Find a file that is 1033 bytes in size, human-readable, and not executable.
* **Methodology:** I iterated through three different approaches to refine the output.
    * **Method 1 (Basic Find):** `find . -size 1033c`
    * **Method 2 (Exec):** Executing cat on the found file immediately.
        ```bash
        find . -size 1033c -exec cat {} \;
        ```
    * **Method 3 (Clean Output):** Printing the filename alongside the content for clarity.
        ```bash
        find . -size 1033c -exec sh -c 'echo "==={}==="; cat "{}"' \;
        ```

---

## Level 7: Permissions and Error Redirection
* **Objective:** Find a file owned by user `bandit7`, group `bandit6`, and size 33 bytes.
* **The Challenge**
	Running a standard find command from the root directory generated a massive amount of "Permission Denied" errors, obscuring the valid result.
* **The Solution**
	I used **Standard Error (stderr) Redirection** to send all error messages to `/dev/null` (the null device), leaving only the valid output.
	
	**Command:**
	```bash
	find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
	```
	
	**Advanced Execution:** To read the file automatically once found:
	```bash
	find / -user bandit7 -group bandit6 -size 33c -exec cat {} \; 2>/dev/null
	```
