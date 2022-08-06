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

Services:
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
* A `mysql` server is running. It is using `MariaDB`. I'm not familiar with MariaDB, but the commands are in a public info document somewhere. I'll find them. 

A quick visit to the Daily Bugle website gives the answer to task 1. Who robbed the bank? Sp[REDACTED]an 

It also shows a login form we can try to abuse. I've captured the home page source for future reference. 

Continuing with our initial recon. 

### Dirb Scan

### Gobuster Scan

### Nikto Scan

### SMBMAP Scan


## Task 1
*Access the web server, who robbed the bank?* Sp[REDACTED]an 

