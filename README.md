# TryHackMe – Corridor (Methodology)

## Objective

The goal of this challenge is to identify and exploit an **Insecure Direct Object Reference (IDOR)** vulnerability by analyzing how the application references objects through URL endpoints.

---

## Step 1 – Port Scanning

Start by identifying the exposed services on the target.

```bash
nmap -F 10.130.129.217
```
<img width="566" height="114" alt="image" src="https://github.com/user-attachments/assets/ef43eee2-21ef-4fd8-88b8-c148cfab2a9b" />


The scan reveals that the target only exposes an HTTP service.

---

## Step 2 – Directory Enumeration

Next, perform a directory enumeration to look for hidden files or directories.

```bash
gobuster dir -u http://10.130.129.217 -w /usr/share/wordlists/dirb/common.txt
```
<img width="367" height="169" alt="image" src="https://github.com/user-attachments/assets/59059458-8331-4d88-9ca4-00365ca302be" />


No interesting directories or files are discovered, suggesting that the challenge likely revolves around the web application's functionality rather than hidden content.

---

## Step 3 – Explore the Website

Visiting the application presents a corridor with multiple doors.

Clicking on different doors changes the URL to hexadecimal values similar to the following:

```
http://10.130.129.217/c4ca4238a0b923820dcc509a6f75849b
http://10.130.129.217/c81e728d9d4c2f636f067f89cc14862c
http://10.130.129.217/eccbc87e4b5ce2fe28308fd9f2a7baf3
```

These values appear to be hashes rather than random identifiers.

---

## Step 4 – Identify the Hash Type

Use **nth** (Name-That-Hash) to identify the hash algorithm.

```bash
nth -t eccbc87e4b5ce2fe28308fd9f2a7baf3
```

<img width="397" height="158" alt="image" src="https://github.com/user-attachments/assets/d746739e-0f3e-4fa0-b6d6-b15bebfe1087" />

The tool identifies the value as an **MD5** hash.

To verify the contents, use an MD5 cracking service such as CrackStation.

The hashes resolve to:

| Hash                               | Value |
| ---------------------------------- | ----- |
| `c4ca4238a0b923820dcc509a6f75849b` | `1`   |
| `c81e728d9d4c2f636f067f89cc14862c` | `2`   |
| `eccbc87e4b5ce2fe28308fd9f2a7baf3` | `3`   |

This reveals that the application is simply using the **MD5 hash of sequential numeric IDs** as object references.

---

## Step 5 – Identify the IDOR

The challenge description hints that the vulnerability is an **IDOR**.

The application exposes resources using:

```
MD5(1)
MD5(2)
MD5(3)
...
```

instead of enforcing proper authorization.

The clue:

> *"Can you find your way back to where you came?"*

suggests navigating to the object before `1`, which is `0`.

---

## Step 6 – Generate the Missing Object ID

Generate the MD5 hash of `0`.

For example:

```bash
echo -n 0 | md5sum
```

or

```python
import hashlib

print(hashlib.md5(b"0").hexdigest())
```

Use the resulting hash as the URL endpoint.

Visiting the corresponding endpoint reveals the challenge flag.

---

## Key Takeaways

* Hashing object identifiers does **not** prevent IDOR vulnerabilities.
* Predictable identifiers remain predictable even when hashed.
* Authorization must always be enforced on the server side.
* Enumerating object references and recognizing common hash formats are valuable skills during web application assessments.
