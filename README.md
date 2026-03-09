# DVWA Security Lab Report

## Application Security Testing Lab

---

# 1. Environment Setup

## 1.1 Install Docker

Verify installation:

```bash
docker --version
```

Output:

```bash
Docker version 29.2.1
```

---

# 2. Deploy DVWA in Docker

## Pull DVWA Image

```bash
docker pull vulnerables/web-dvwa
```

## Run DVWA Container

```bash
docker run -d \
--name dvwa \
-p 8080:80 \
vulnerables/web-dvwa
```

## Verify Container

```bash
docker ps
```

Example output:

```bash
CONTAINER ID   IMAGE                  COMMAND      CREATED         STATUS         PORTS                                     NAMES
94f3c45d157b   vulnerables/web-dvwa   "/main.sh"   3 minutes ago   Up 3 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   dvwa
```

Open in browser:

```
http://localhost:8080
```

---

# 3. Vulnerability Testing

Security levels tested:

* Low
* Medium
* High

---

# 3.1 SQL Injection

## Security Level: Low

Payload Used:

```
1' OR '1'='1
```

Result:

All user records were displayed.

Screenshot:

![SQL Injection Low](images/sql/low.png)


Explanation:

At the Low security level, user input is directly inserted into the SQL query without input validation or sanitization. The payload modifies the SQL query logic so the condition always evaluates to true.

Query:

```sql
SELECT * FROM users WHERE id='1' OR '1'='1';
```

This causes the database to return all records.

---

## Security Level: Medium

Payload Used:

```
1/2/3/4/5
```

Result:

User records were still displayed.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The Medium level uses `mysql_real_escape_string()` to escape quotes and special characters. However, the input parameter is numeric and not enclosed in quotes in the SQL query. Because of this, escaping does not prevent SQL manipulation.

The query remains vulnerable because the developer relied only on input escaping rather than parameterized queries.

---

## Security Level: High

Payload Used:

```
a' UNION SELECT "test1","test2";-- -
```

Result:

Custom values appeared in the result.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The High security level introduces additional checks such as session validation. However, the SQL query is still constructed dynamically and does not use prepared statements. This allows attackers to use UNION-based SQL injection to modify the query result and inject custom values.

---

# 3.2 Reflected XSS

## Security Level: Low

Payload Used:

```
<script>alert('XSS')</script>
```

Result:

A JavaScript alert popup appeared.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

At the Low security level, user input is reflected directly in the web page without sanitization. Because the input is inserted into the HTML output, the browser interprets it as executable JavaScript.

---

## Security Level: Medium

Payload Used:

```
<ScRiPt>alert("XSS")</ScRiPt>
```

Result:

Alert popup appeared.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The Medium level attempts to filter `<script>` tags, but the filter is case-sensitive. By changing the letter casing of the tag, the attacker bypasses the filter and executes JavaScript.

---

## Security Level: High

Payload Used:

```
<img src="invalid.jpg" onerror="alert(1)">
```

Result:

Alert popup appeared.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The filter removes `<script>` tags but does not sanitize HTML event attributes such as `onerror`. By injecting an image element with an event handler, JavaScript execution is triggered when the image fails to load.

---

# 3.3 Stored XSS

## Security Level: Low

Payload Used:

Name:

```
Muskan
```

Message:

```
<script>alert('Stored XSS')</script>
```

Result:

The alert appeared every time the page reloaded.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The application stores user input directly in the database without sanitization. When the page loads, the stored script executes automatically.

---

## Security Level: Medium

Payload Used:

```
<sCriPt>alert("XSS");</sCriPt>
```

Result:

Alert appeared.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The Medium level introduces partial filtering, but not all fields are protected. The Name field does not properly sanitize input, allowing JavaScript code to be stored and executed.

---

## Security Level: High

Payload Used:

```
</div><img src=x onerror=alert(1)>
```

Result:

The payload did not execute JavaScript.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

At the High security level, the application performs stronger input filtering and output encoding. Dangerous attributes are removed and special characters are encoded. Because of this, the browser treats the input as plain text rather than executable JavaScript.

---

# 3.4 Command Injection

## Security Level: Low

Payload Used:

```
127.0.0.1 && dir
```

Result:

The directory listing was displayed after the ping command executed.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The application executes system commands directly using user input. By appending `&&`, attackers can run additional commands after the original command completes.

---

## Security Level: Medium

Payload Used:

```
127.0.0.1 & dir
```

Result:

The ping command ran in the background and the directory listing was displayed.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

Some command operators are filtered, but the background operator `&` is not blocked. This allows attackers to execute additional commands.

---

## Security Level: High

Payload Used:

```
127.0.0.1|dir
```

Result:

Both ping output and directory listing were displayed.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The developer attempted to filter input using pattern matching and `trim()`. However, `trim()` only removes whitespace from the beginning and end of the input. Removing spaces around the pipe operator allows the attacker to bypass the filter.

---

# 3.5 Blind SQL Injection

## Security Level: Low

Payload Used:

```
1' AND 1=1#
```

Result:

The message **"User ID exists in the database"** appeared.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

At the Low level, user input is directly inserted into the SQL query without validation. Logical conditions such as `AND 1=1` allow attackers to manipulate the query and confirm vulnerabilities.

---

## Security Level: Medium

Payload Used:

```
1
```

Result:

User ID exists in the database.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

The Medium level uses `mysql_real_escape_string()` but does not enclose the parameter in quotes. Because the value is numeric, escaping does not protect the query from manipulation.

---

## Security Level: High

Payload Used:

```
1' AND SLEEP(5)#
```

Result:

The webpage response was delayed by approximately 5 seconds.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

Although database errors are hidden, injected SQL commands are still executed. Using the `SLEEP()` function creates a delay, allowing attackers to confirm the vulnerability through timing differences.

---

# 3.6 JavaScript Attacks

## Security Level: Low

Payload Used:

```
success + generate_token()
```

Result:

Token updated and validation succeeded.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

Token generation is implemented entirely in client-side JavaScript. By analyzing and manually executing the token function in the browser console, attackers can bypass validation.

---

## Security Level: Medium

Payload Used:

```
do_elsesomething("XX")
```

Result:

Token regenerated successfully.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

Token generation still occurs in client-side JavaScript. Attackers can execute the required functions manually in the console to generate valid tokens.

---

## Security Level: High

Result:

Token could not be regenerated manually.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

Token validation is moved to the server side. Since the browser no longer controls token generation, manually executing JavaScript functions cannot bypass the validation mechanism.

---

# 3.7 Brute Force

## Security Level: Low

Payload Used:

```
admin : password
```

Result:

Login successful.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

There are no protections such as rate limiting, CAPTCHA, or account lockout. Attackers can attempt unlimited login combinations.

---

## Security Level: Medium

Payload Used:

```
admin : password
```

Result:

Login successful.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

A delay is introduced after failed login attempts. This slows brute force attacks but does not prevent them.

---

## Security Level: High

Payload Used:

```
admin : password
```

Result:

Authentication successful.

Screenshot:

```
[Insert Screenshot Here]
```

Explanation:

Additional protections such as CSRF tokens are implemented. However, weak credentials still allow attackers to gain access once discovered.

---

# 4. Docker Inspection

## Running Containers

```bash
docker ps
```

---

## Inspect Container

```bash
docker inspect dvwa
```

This command shows detailed container configuration including networking, environment variables, and mounted volumes.

---

## Container Logs

```bash
docker logs dvwa
```

Logs show application activity and server events.

---

## Access Container Shell

```bash
docker exec -it dvwa /bin/bash
```

Inside the container:

```bash
ls /var/www/html
```

Example output:

```
config
dvwa
hackable
index.php
```

---

# 5. Application Architecture Analysis

## Where Application Files Are Stored

Application files are located in:

```
/var/www/html
```

This is the default web root directory used by Apache web servers.

---

## Backend Technology Used

DVWA uses:

* PHP for server-side logic
* MySQL database
* Apache web server

---

## How Docker Provides Isolation

Docker isolates the DVWA environment by running it inside a container. The container includes its own filesystem, dependencies, and runtime environment. This prevents the vulnerable application from affecting the host system and allows safe testing.

---

# 6. Security Analysis

## Why SQL Injection Succeeds at Low Security

At the Low security level, user input is inserted directly into SQL queries without validation or sanitization. Because the application dynamically constructs SQL queries, attackers can modify the query logic.

---

## What Control Prevents It at High Security

The most effective control is the use of **prepared statements (parameterized queries)**. These separate SQL code from user input, preventing injected commands from being executed.

---

## Does HTTPS Prevent These Attacks?

No.

HTTPS encrypts data during transmission but does not prevent vulnerabilities such as SQL injection or cross-site scripting. These attacks exploit weaknesses in server-side application logic rather than network communication.

---

## Risks if the Application is Publicly Accessible

If deployed publicly, attackers could:

* Steal sensitive data
* Execute malicious scripts
* Gain unauthorized system access
* Modify or delete database records
* Compromise user accounts

---

# 7. OWASP Top 10 Mapping

| Vulnerability       | OWASP Category                                  |
| ------------------- | ----------------------------------------------- |
| SQL Injection       | A03: Injection                                  |
| Blind SQL Injection | A03: Injection                                  |
| Command Injection   | A03: Injection                                  |
| Reflected XSS       | A03: Injection                                  |
| Stored XSS          | A03: Injection                                  |
| Brute Force         | A07: Identification and Authentication Failures |
| JavaScript Attacks  | A05: Security Misconfiguration                  |

These vulnerabilities demonstrate common weaknesses described in the **OWASP Foundation Top 10 security risks.

---

