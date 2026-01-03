# Basmach Hanukkah 2025 Cyber Challenge – Solution

This document describes my solution to the Hanukkah 2025 cybersecurity challenge published by Basmach.

## Context and Timing

This challenge was solved **during the official time window**, before any solutions or walkthroughs were published.
After the challenge ended, it was officially announced that **95 participants successfully completed it**, and my solution was one of them.
Only later were public solutions and video walkthroughs released.

This write-up documents my original reasoning process and the technical steps I followed while solving the challenge independently.

---

## Stage 1 – Identifying the Cipher

The first step was to understand the meaning of each line in the initial text and how it could help identify the encryption method behind the final string:

FCvkQ#c(U]h7p.+W2?1^Wc4RR

After reading the text several times, I concluded that it referred to a cipher code and to New York City, specifically the buildings between streets.
New York is structured as a grid, with streets and avenues forming a clear layout.

Using Google Maps, I marked the locations described in the text.
At each intersection, I noticed that the building shapes resembled Roman letters.
Combining these letters resulted in the number 92.

Based on this observation, I attempted to decode the string using Base92 encoding.
This successfully revealed a URL leading to the next stage of the challenge:

goramli.bsmch.idf.il

---

## Stage 2 – Hidden File Extraction

The website displayed an Arabic sentence:

"من برّا رخام ومن جوّا سخام"

Which translates to:
"Marble on the outside, soot on the inside."

This hinted that something was hidden beneath the surface.
Inspecting the page revealed a hidden file named `hidden_image.png`.

At first glance, the image appeared to be a simple green square.
However, running `strings` on the file revealed that it contained an embedded executable.

To extract it, I searched for the PE header (`MZ`) using:

grep -a -b -o "MZ" secret_image.exe | head -n 5

This showed that the executable started at offset 3996.
I then extracted the executable using:

dd if=secret_image.exe of=real_app.exe bs=1 skip=3996

---

## Stage 3 – Executable Analysis and Alternate Data Streams

Running the extracted executable initially failed due to a missing file:

C:\Message\Secret.txt

I created the file at the specified path and ran the executable again.
This time it executed successfully and presented three riddles that needed to be solved in order to proceed.

The riddles were:

**Riddle 1:**
"Tell me a secret and I'll let you in,  
Prove who you are, and your access begins.  
Without the right claim, the door stays locked tight,  
For I verify truth before granting the right.  
Who am I?"

Answer: **Authentication**

**Riddle 2:**
"I turn names into numbers so machines know the way,  
Poison my tables and you'll go astray.  
I guide your browser's footsteps through the 'World-Wide' mess,  
A phonebook for the internet, no more, no less.  
Who am I?"

Answer: **DNS**

**Riddle 3:**
"Run code in my walls, yet the system won't fall,  
I'm a safe little cage that contains it all.  
Kids play with me, but this ain't no fun nor games,  
For if something breaks out, all will go up in flames.  
Who am I?"

Answer: **Sandbox**

After answering all three correctly, the program instructed me to combine the first letters of the answers, resulting in:

ADS

This refers to **Alternate Data Streams**, a Windows feature that allows additional data to be hidden within a file.

I checked for alternate streams attached to `Secret.txt` using:

Get-Item -Path C:\Message\Secret.txt -Stream *

This revealed two additional streams:
- DEC_ALG  
- ENC_URL  

Reading them using:

Get-Content -Path C:\Message\Secret.txt -Stream DEC_ALG  
Get-Content -Path C:\Message\Secret.txt -Stream ENC_URL  

Indicated that I needed to decode the string:

H9:E6=:@?]3D>49]:57]:=

Using the ROT47 cipher.

Decoding it produced another URL:

whitelion.bsmch.idf.il

---

## Stage 4 – Web Exploitation

Visiting the new website displayed an image of a robot.
This led me to check the site's `robots.txt` file:

https://whitelion.bsmch.idf.il/robots.txt

There, I found the first flag:

{narniabale}

The file also disallowed access to `/login.php`.
Navigating to this page revealed a login form.

After several attempts, I tried a basic SQL injection payload in the username field:

' OR 1=1 --

This successfully bypassed authentication and granted access to the site.

Immediately after logging in, I noticed the second flag:

{bsmch}

The site described four types of attacks.
Three were accessible, while the fourth was marked as classified.

Observing the URL structure showed that tasks were accessed using an `id` parameter (e.g., `?id=1`).
I replaced the value with `4`:

https://whitelion.bsmch.idf.il/task.php?id=4

This revealed the third flag:

{idf}

The page also instructed to elevate user permissions.
Inspecting the cookies showed a `role` value.
Changing it to `admin` and refreshing the page revealed the final flag:

{il}

---

## Final Result

Combining all collected flags resulted in the final URL of the challenge:

https://narniabale.bsmch.idf.il/

---

## Summary

This challenge required combining multiple domains:
- Cipher identification and decoding
- File and binary analysis
- Windows Alternate Data Streams
- Web security basics (robots.txt, SQL injection, IDOR, cookie manipulation)

The solution emphasized reasoning, pattern recognition, and practical use of common security tools rather than advanced exploitation techniques.
