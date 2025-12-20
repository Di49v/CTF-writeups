# Bandit Wargame: Levels 11 -> 13

**Date:** 2025-12-20
**Difficulty**: Medium
**Key Concepts:** [[tr]], [[tar]], [[gzip]], [[gunzip]], [[bzip2]], [[bunzip2]]

---
### Level 11 -> 12
* **Objective:** The password for the next level is stored in the file **data.txt**, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.
* **Solution:**
    ```bash
    cat data.txt | tr 'a-zA-Z' 'n-za-mN-ZA-M'
    ```
* **Key Learning:** Learnt the `tr` command for `rot13`. Earlier, I was thinking about writing a code for it as I didn't know about this command but as it turns out, for simple tasks like ROT13 because Linux has powerful built-in tools that can solve these in a single line.

---

### Level 12 -> 13

- **Objective:** The password for the next level is stored in the file **data.txt**, which is a hexdump of a file that has been repeatedly compressed.
- **Difficulty:** Medium (if you use the `file` command), but I did it the **Hard Way** using manual analysis and deduction.
* ***Solution:**
	This level was a massive learning experience. It required understanding file systems, hexadecimal reversal, and the "signatures" left behind by different compression algorithms. Because I was strictly following _The Linux Command Line_ (TLCL) and avoiding spoilers, I had to figure out what type of file I was looking at by analyzing file sizes and raw content patterns.

	#### 1. Creating a Workspace
	
	First, I needed a place to work. You cannot create files in the home directory on Bandit, so I had to use `/tmp/`. It's important to remember that `/` is the root of the system, not your home folder. I used `mktemp -d` to create a secure, hidden directory so other players wouldn't interfere with my files.
	
	Bash
	
	```
	# Generate a hard-to-guess name and store it in a variable
	myfile=$(mktemp -d /tmp/tislevel.XXXXXXXX)
	cd $myfile
	cp ~/data.txt .
	```
	
	#### 2. Reversing the Hexdump
	
	The prompt mentioned `data.txt` was a hexdump. While `xxd` isn't actually in the TLCL book, the challenge description made it clear that converting to a hexdump was the very last thing the creator did. Therefore, reversing it had to be my first step.
	
	Bash
	
	```
	xxd -r data.txt > data.txt.revxxd
	```
	
	#### 3. The "Read and Think" Decompression Marathon
	
	This is where the "hard way" began. Without the `file` command, I had to guess the compression type, try a tool, and observe the result.
	
	The Realization about File Sizes:
	
	I noticed a fascinating pattern. data.txt was the largest (ASCII hex is bulky). My reversed binary was smaller. The first gunzip made it smaller, and the following bunzip2 made it smaller still. However, the next gunzip resulted in the largest binary file yet.
	
	This told me something about the person who made the level: they likely compressed it once with `gzip` (making it exponentially smaller), but then further attempts to compress it with `bzip2` and `gzip` actually _increased_ the size because the data was already "maxed out." The extra headers just added bloat.
	
	The Step-by-Step Grind:
	
	I had to rename files constantly because tools like gunzip refuse to work unless the file has the correct extension (like .gz).
	
	Bash
	
	```
	# Layer 1: Gzip
	cp data.txt.revxxd gcomp1.txt.gz
	gunzip gcomp1.txt.gz
	
	# Layer 2: Bzip2 (I tried gzip first, it failed, so I switched to bzip2)
	cp gcomp1.txt bcomp2.txt.bz2
	bunzip2 bcomp2.txt.bz2
	
	# Layer 3: Gzip again
	cp bcomp2.txt gcomp3.txt.gz
	gunzip gcomp3.txt.gz
	```
	
	#### 4. Discovering the Tar Archive
	
	After those layers, `cat` showed me something new. I saw the string `ustar` and filenames like `data5.bin`. From my reading, I knew `ustar` meant a **tar** archive.
	
	Bash
	
	```
	# Layer 4: Tar
	cp gcomp3.txt tcomp.tar
	mkdir data
	cd data
	tar xf ../tcomp.tar
	
	# This resulted in data5.bin. I saw it was another tar.
	cp data5.bin tcomp2.tar
	tar xf tcomp2.tar
	
	# This resulted in data6.bin. 
	# I noticed gz and bz2 were being used alternately.
	# Since the last one was gunzip, I tried bz2 directly.
	mv data6.bin ../bcomp4.txt.bz2
	cd ..
	bunzip2 bcomp4.txt.bz2
	```
	
	#### 5. The Final Stretch
	
	I checked the content of `bcomp4.txt` and saw the familiar `ustar` pattern again, mentioning a `data9.bin`.
	
	Bash
	
	```
	# Layer 5: Another Tar
	cp bcomp4.txt tcomp3.tar
	cd data
	tar xf ../tcomp3.tar
	
	# This gave me data8.bin. I suspected one last gzip.
	cp data8.bin ../gcomp5.txt.gz
	cd ..
	gunzip gcomp5.txt.gz
	
	# Final Reveal
	cat gcomp5.txt
	```
	
	#### 6. Epilogue: The "Better Way"
	
	After finally getting the password, I felt like a real hacker for recognizing those patterns manually. However, I eventually googled it and realized there is a much "better" way: the `file` command.
	
	If I had used `file data.txt.revxxd`, the system would have told me exactly: "gzip compressed data." I wouldn't have had to guess or look for `ustar` strings. While my "Read and Think" method was adventurous and taught me how headers and compression overhead work, using `file` turns this from a hard level into a medium one.
	
	**Key Learning:** Real hacking is about persistence and observation, but knowing your tools (like `file`) makes you much more efficient!
	