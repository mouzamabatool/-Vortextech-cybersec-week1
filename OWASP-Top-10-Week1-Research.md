# OWASP Top 10 Vulnerabilities — Week 1 Research
**Cyber Security Internship Track — Week 1 of 4**

This document covers five vulnerabilities from the current **OWASP Top 10 (2025 edition)**, explained in plain language, with an example of how each could be exploited, a real-world case, and a practical prevention tip for developers.

> Note on the list version: OWASP released an updated Top 10 in late 2025 (the previous version was from 2021). A few categories were renamed or merged — for example, Server-Side Request Forgery was folded into Broken Access Control, and two new categories (Software Supply Chain Failures and Mishandling of Exceptional Conditions) were added. The five vulnerabilities below are still the same core issues under their current 2025 names.

---

## 1. Broken Access Control (A01:2025)

### What it is
Access control is basically the rulebook that decides who is allowed to see or do what in an application — for example, only you should be able to view your own bank statement, not your neighbor's. "Broken" access control means those rules aren't properly enforced, so a user can end up doing things or viewing data they were never supposed to have access to.

### How an attacker could exploit it
A very common version of this is called an **Insecure Direct Object Reference (IDOR)**. Imagine a website URL like:

```
https://example.com/account?id=1001
```

If an attacker simply changes the number to `id=1002` and the server doesn't check whether that account actually belongs to them, they can view (or sometimes edit) someone else's account. No hacking tools required — just changing a number in the address bar. Other examples include a regular user finding and using an admin-only page URL directly, or a mobile app's "hidden" API endpoints being called directly with a different user's ID.

### Real-world example
This category is consistently ranked the #1 or #2 risk in OWASP's data because it shows up in an enormous share of tested applications, and IDOR-style bugs have been responsible for numerous large data exposures — including well-documented cases where researchers or attackers were able to pull other customers' records from apps (banking, ride-share, and social media platforms have all had public IDOR incidents) simply by manipulating an ID in a request.

### How to prevent it
- **Enforce access control checks on the server for every request**, not just in the app's menus or buttons. The server should verify "does this logged-in user actually own this record / have this role?" before returning any data — never trust that the user won't tamper with a URL or request.

---

## 2. Injection (A05:2025)

### What it is
Injection happens when an application takes input from a user (a search box, a login form, a URL parameter) and passes it directly into a command that gets executed — like a database query — without checking it first. If the input contains malicious code disguised as "data," the application ends up running instructions the attacker wrote, not just the data it expected.

### How an attacker could exploit it
The classic example is **SQL Injection**. Suppose a login form builds a database query like this behind the scenes:

```
SELECT * FROM users WHERE username = 'INPUT' AND password = 'INPUT'
```

If the developer just drops the raw text a user types into that query, an attacker can type something like `' OR '1'='1` into the username field. The query becomes a statement that is always true, and the attacker logs in without knowing any real password. More advanced attackers can use similar tricks to dump an entire database of usernames, passwords, and credit card numbers.

### Real-world example
The 2017 **Equifax breach**, which exposed the personal data of roughly 147 million people, is often cited alongside this category of vulnerability — though in that specific case the root cause was an unpatched web framework flaw rather than classic SQL injection. SQL injection itself has a long track record of causing major breaches, including the 2008–2009 **Heartland Payment Systems** breach, where attackers used SQL injection to install malware and steal over 130 million credit card numbers, at the time the largest breach of its kind in U.S. history.

### How to prevent it
- **Use parameterized queries (prepared statements)** instead of building queries by gluing together strings of user input. This tells the database "treat this input strictly as data, never as part of the command," which closes off the injection trick regardless of what the attacker types.

---

## 3. Security Misconfiguration (A02:2025)

### What it is
This isn't a flaw in the application's code — it's a mistake in how the application, server, database, or cloud service is *set up*. Things like leaving default admin passwords unchanged, leaving debugging features turned on in production, leaving unnecessary features/ports enabled, or misconfiguring cloud storage permissions all fall under this umbrella.

### How an attacker could exploit it
A very common real scenario: a company stores files in a cloud storage bucket (like an AWS S3 bucket) and forgets to restrict who can access it. An attacker doesn't need to "hack" anything technical — they can simply browse to the storage bucket's public web address and download whatever is inside, because the permissions were never locked down. Similarly, leaving a database's default admin login (like `admin/admin`) unchanged lets an attacker walk right in.

### Real-world example
Numerous companies have had customer data exposed simply because a cloud storage bucket was left open to the public with no password or access restriction — a mistake that requires no advanced hacking skill to exploit, just knowing (or guessing) the storage location's address. This kind of misconfiguration has affected organizations across finance, telecom, and government sectors over the years.

### How to prevent it
- **Harden and review configurations before going live**, using a checklist: disable default accounts, turn off debug/verbose error messages in production, apply the principle of least privilege to storage and cloud permissions, and regularly re-scan environments for configuration drift (settings quietly changing back to insecure defaults over time).

---

## 4. Authentication Failures (A07:2025)

### What it is
This covers weaknesses in how an application confirms "you are who you say you are." It includes things like allowing extremely weak or common passwords, not limiting how many login attempts someone can make, poorly protecting session tokens (the digital "pass" that keeps you logged in), or not offering multi-factor authentication (MFA).

### How an attacker could exploit it
A common attack here is **credential stuffing**: attackers take huge lists of usernames and passwords leaked from *other* past breaches and automatically try them against a different website, betting that some people reused the same password. If the site doesn't limit repeated login attempts or require MFA, a bot can try thousands of combinations per minute until some work. Another version is **brute forcing** a login form that has no lockout after failed attempts.

### Real-world example
Credential stuffing attacks have hit major consumer platforms repeatedly — streaming services, retailers, and food-delivery apps have all reported incidents where attackers used lists of leaked username/password pairs from unrelated breaches to break into accounts that reused the same password, sometimes making fraudulent purchases or accessing stored payment details.

### How to prevent it
- **Require multi-factor authentication (MFA) and rate-limit or lock out repeated failed login attempts.** Even if an attacker has a valid password from a leaked list, MFA stops them from getting in, and rate-limiting makes automated guessing impractical.

---

## 5. Cryptographic Failures (A04:2025)

### What it is
This is about sensitive data — passwords, credit card numbers, health records, personal identifiers — not being properly protected with encryption, either while it's stored (**at rest**) or while it's traveling across the internet (**in transit**). This includes using outdated/weak encryption algorithms, or in the worst cases, storing sensitive data in plain, readable text with no encryption at all.

### How an attacker could exploit it
If a company stores customer passwords as plain, unencrypted text in its database, then anyone who manages to break into that database — through any other vulnerability — instantly has every user's real password, no extra effort required. Similarly, if a website doesn't use HTTPS properly, an attacker on the same public Wi-Fi network can intercept login credentials or payment details as they're being typed and sent.

### Real-world example
The 2013 **Adobe breach** exposed encrypted passwords for roughly 150 million user accounts, but because Adobe had used a weak, outdated encryption method (rather than a strong modern password-hashing approach) along with unencrypted password hints, security researchers were able to crack a large number of the passwords relatively easily — turning what should have been a survivable breach into a much bigger exposure of real passwords.

### How to prevent it
- **Never store passwords in plain text or with weak/outdated encryption — use a strong, modern password-hashing algorithm (like bcrypt or Argon2), and enforce HTTPS/TLS everywhere data travels**, so that even if attackers steal the data, it isn't immediately usable.

---

## Personal Reflection

The vulnerability that surprised me most was **Security Misconfiguration**. I expected the most damaging breaches to come from clever, highly technical attacks like custom exploit code or advanced injection techniques, but it turns out that something as simple as an unlocked cloud storage bucket or an unchanged default password has caused some of the largest data exposures on record. It's a good reminder that in security, the basics — careful setup, regular review, and not leaving default settings in place — matter just as much as defending against sophisticated attackers.

---

## Sources Consulted
- OWASP Top 10 official project (owasp.org/Top10)
- OWASP Top 10:2025 category list and release notes
- Public reporting on the Equifax (2017), Heartland Payment Systems (2008–2009), and Adobe (2013) breaches
- General industry writeups on credential stuffing and IDOR-style access control failures
