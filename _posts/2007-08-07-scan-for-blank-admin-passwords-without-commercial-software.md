---
title: "Scan for Blank Admin Passwords without Commercial Software"
date: "2007-08-07"
layout: post
---

I've seen blank administrator passwords at every organization I've worked. Without fail there will be some user that manages to get a PC onto your network without setting a password. This type of scenario opens up Pandora's box into the number of vectors that could be created. Once a malicious user has control over a machine on your network its essentially game over. So as someone with security and risk management in mind you want to periodically scan for such activity, but your organization isn't gonna spring for some fancy tool. Luckily this task can be put into a windows script that can check for this condition, see the script below.


```bat
**On Error Resume Next
Set objNetwork = CreateObject("Wscript.Network")
strComputer = objNetwork.ComputerName
strPassword = ""
Set colAccounts = GetObject("WinNT://" & strComputer)
colAccounts.Filter = Array("user")
For Each objUser In colAccounts
    objUser.ChangePassword strPassword, strPassword
    If Err = 0 or Err = -2147023569 Then
        Wscript.Echo objUser.Name & " is using a blank password."
    End If
    Err.Clear
Next
```

I can't take credit for this script, the "Scripting Guy" has a nice article better explaining this script [here](http://www.microsoft.com/technet/scriptcenter/resources/qanda/oct05/hey1006.mspx "Scan blank admin password script"). This simple script will let you find blank administrator passwords just like those expensive commercial scanners. The commercial scanners on the market may not use this exact script but do use the same principle when scanning for blank administrator passwords. Now that you have found PC's with blank administrator passwords you will need to verify the findings of your script by connecting to that PC. The steps laid out below for connecting to a machine may seem simplistic but there are numerous users out there that aren't familiar with this process. If they were they wouldn't have blank administrator passwords now would they.

You could just map to a drive on that machine and browse files that are on that drive. When mapping to a drive it needs to be in the form "\\\\computer name\\drive letter". For example if I was trying to connect to drive C: on a computer named "Pam" it would look like Figure 1 below.

![](/assets/mapnetworkdrive.bmp)

Figure 1: Map to drive on PC with blank admin password

Once you hit finish it will try and connect using your username or the username you are currently logged in with by default. If the username does not exist it will prompt you for a username and password. For a blank administrator password just enter administrator as the username and nothing for the password.

![](/assets/connecttomachine.bmp)

Figure 2: Username and password prompt

You should now see the contents of the C: drive on the computer named "Pam". You could also enter the IP address instead of the computer name, either way works. This connection can also be made via the "net use" command. This command can be seen in Figure 3.

![](/assets/connectusingcommandline.bmp)

Figure 3: Connect to PC with blank admin password via command line

The asterisk after the username "administrator" will trigger a prompt for the password, which in this case you will just hit enter because the password is blank. Verification of these types of vulnerabilities will show others within your organization the seriousness of users not having strong passwords. Especially when you include screen shots showing the local hard drive of another user on the network. So just because you don't have a fancy commercial scanner doesn't mean you can't periodically review your network. Running this type of script would also be a great way to correlate some of your findings with other tools that you may have at your disposal. Happy hunting on your network.
