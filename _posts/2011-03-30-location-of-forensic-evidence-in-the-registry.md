---
title: "location of forensic evidence in the registry"
date: "2011-03-30"
layout: post
---

I got tired of always searching online for the location of something in the windows registry, especially when it came to forensic analysis. Hopefully this compilation will help others to find things of interest inside the windows registry. My plan is to update this article as I find more interesting locations within the windows registry.


Last logged on user

```bat
HKLM\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\WinLogon

DefaultUserName
```


Searches within the windows OS

```bat
HKCU\\Software\\Microsoft\\Search Assistant\\ACMru
5001: Contains list of terms used for the internet search assistant
5603: Contains the list of terms used for the Windows XP files and folders search
5604: Contains list of terms used in the “word or phrase in a file” search
5647: Contains list of terms used in the “for computers or people” search
```

Applications launched from the "Start > Run" menu

```bat
HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\RunMRU
```

Recent documents

```bat
HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\RecentDocs
```

Installed applications that reside in "Control Panel > Add/Remove programs"

```bat
HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall
```

Mounted devices

```bat
HKLM \\SYSTEM\\MountedDevices
```

```bat
HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\MountPoints2\\CPC\\Volume\\
```

USB devices that have been attached

```bat
HKLM\\SYSTEM\\CurrentControlSet\\Enum\\USBSTOR
```

Applications that are ran during startup

```bat
HKLM\\SOFTWARE \\Microsoft\\Windows\\CurrentVersion\\Run
HKLM\\SOFTWARE \\Microsoft\\Windows\\CurrentVersion\\RunOnce
HKLM\\SOFTWARE \\Microsoft\\Windows\\CurrentVersion\\RunOnceEx
HKLM\\SOFTWARE \\Microsoft\\Windows\\CurrentVersion\\RunServices
HKLM\\SOFTWARE \\Microsoft\\Windows\\CurrentVersion\\RunServicesOnce
HKLM\\System\\CurrentControlSet\\Control\\Session Manager\\BootExecute
```

List of windows services

```bat
HKLM\\SYSTEM\\CurrentControlSet\\Services\\
```

Recent network settings, where GUID refers to the network interface

```bat
HKLM\\SYSTEM\\CurrentControlSet\\Services\\Tcpip\\Parameters\\Interfaces\\GUID
```

Wireless network information

```bat
HKLM\\SOFTWARE\\Microsoft\\WZCSVC\\Parameters\\Interfaces\\GUID
```

Mapped network drives

```bat
HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Map Network Drive MRU
```

Typed URL's into the browser

```bat
HKU\\UID\\Software\\Microsoft\\Internet Explorer\\TypedURLs
```

Last time the computer was shut down (64bit value representing time)

```bat
HKLM\\SYSTEM\\CurrentControlSet\\Control\\Windows
```

Determine if last access times is enabled (0) or disabled (1)

```bat
HKLM\\System\\CurrentControlSet\\Control\\FileSystem\\

NtfsDisableLastAccessUpdate
```

Computer name

```bat
HKLM\\System\\CurrentControlSet\\Control\\ComputerName
```

Determine if autoplay is disabled / enabled, link with more info below

http://support.microsoft.com/kb/967715

```bat
HKU\\UID\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\Explorer\\NoDriveTypeAutoRun
```

List of files open or saved via windows explorer

```bat
HKU\\UID\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\ComDlg32\\OpenSaveMRU
```

List of drives mapped via the map network drive wizard

```bat
HKU\\UID\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Map Network Drive MRU
```

Devices or IP's connected to

```bat
HKU\\UID\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\ComputerDescriptions
```

Mounted drives

```bat
HKU\\UID\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\MountPoints2
```

List of files played in Windows Media Player

```bat
HKU\\UID\\Software\\Microsoft\\MediaPlayer\\Player\\RecentFileList
HKU\\UID\\Software\\Microsoft\\MediaPlayer\\Player\\RecentURLList
```


List of recently accessed WinZip files

```bat
HKU\\UID\\Software\\Nico Mak Computing\\WinZip\\filemenu
```

List of Microsoft Office files that have been accessed

```bat
HKU\\UID\\Software\\Microsoft\\Office\\"version"\\"product"\\File Name MRU
```

Browser helper objects (BHO's), can be associated with malware but it's been a while since I've seen this.

```bat
HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Browser Helper Objects\\
```

Entries in this location are automatically started when explorer.exe is ran

```bat
HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\SharedTaskScheduler\\
```

Can point to logon scripts

```bat
HKLM\\Software\\Policies\\Microsoft\\Windows\\System\\Scripts\\
```

DLL's in this location are loaded when a GUI app is launched

```bat
HKLM\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows\\AppInit\_DLLs
```

Programs to be run when user logs in

```bat
HKLM\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Winlogon\\UserInit
```
