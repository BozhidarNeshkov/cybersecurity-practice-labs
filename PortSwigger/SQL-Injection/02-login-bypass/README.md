# SQL Injection – Login Bypass (PortSwigger)

**Category:** SQL Injection  
**Difficulty:** Apprentice  
**Lab:** SQL injection vulnerability allowing login bypass

## 🧠 Objective
Bypass the login mechanism by exploiting an SQL injection vulnerability in the `username` parameter, allowing authentication as the `administrator` user without knowing the password.

## 📝 Vulnerability Summary
The application processes login requests using an SQL query similar to:

SELECT * FROM users WHERE username = '<USER_INPUT>' AND password = '<PASSWORD>';

The `username` parameter is concatenated directly into the SQL query without proper sanitization, making it vulnerable to SQL injection. By injecting an SQL comment, it is possible to bypass password verification entirely.

## 🎯 Exploitation
I intercepted the login request using Burp Suite Proxy by submitting dummy credentials (`a:a`). This provided a valid CSRF token and session cookie.

### 🔹 Payload used:
administrator'--

The trailing space after `--` is required so the SQL comment is recognized correctly.

### 🔹 Final request body:
csrf=L7IAmuXRsmi3JJrNCHYZ9hPwaWC5HaE2&username=username=administrator'-- &password=anything

### 🔹 Resulting SQL logic (conceptual):
SELECT * FROM users WHERE username = 'administrator'-- ' AND password = 'anything';

The comment sequence (`-- `) removes the password check entirely, allowing login as the administrator user.

## ✅ Result
The server responded with:

HTTP/2 302 Found  
Location: /my-account?id=administrator

This confirmed successful authentication as the administrator user. The browser was redirected to the admin account page, and the lab was marked as solved.
