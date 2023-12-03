---
title: "Security testing iPhone - local data storage"
date: "2012-10-13"
layout: post
---

One of the areas you need to focus on when performing security / penetration testing on iOS applications is what information is written to disk or stored locally. There are a number of things that can be written to disk (text files, config files, plist files, databases, etc). There are a handful of directories that an application typically uses to store local data within an iOS device which you'll need to keep in mind when combing through the local file system. I'll cover these directories and walk through an iOS application security assessment.


The application we'll walk through is iGoat, an intentional vulnerable application written for the iPhone / iOS devices. The iGoat application can be ran inside the iOS Simulator which is free to use assuming you have an Apple OSX device. I'm not going to walk through setting up the iGoat application within the simulator, please check online resources for getting iGoat up and running. Once up and running go to "Data Protection (Rest)" > "Local Data Storage" and start the exercise. You'll notice the exercise describes that it's using an underlying SQLite database to store information, keep in mind you won't normally know this information. Initially when performing a security test on an iOS application you won't know if data is being stored locally, it could be it's storing all information server side. If it does store data locally it can be in a number of locations and in different formats such as a SQLite database. Once you start the iGoat data storage exercise you should come to the following screen.

![](/assets/iGoatLogin.png)

At this point just put in some fake username and password, I'll choose username "super" and password "secret". After you click "Login" nothing will happen, this isn't meant to be a full blown application. So the application took input from the user but what happened to that information? The iOS simulator is no substitution for an iOS device but the behavior of the application remains the same, meaning if you're testing an application in the simulator on your Macbook then if the application stores local data then that data will be stored in some location on your Macbook. The application data is typically stored in the following location on your OS X device.

```
/Users/travis/Library/Application Support/iPhone Simulator/5.1/Applications/UID
```

Where UID is a unique identifier for the application installed and of course you'll substitute my name with your username in the "Users" directory. My UID in this case is "8813AE1B-3FC7-4503-99D0-666E974A74D7". If you open up Finder in OS X and try to navigate to the Library directory you won't see it that's because it's hidden, you can enable the viewing of hidden files and folders but I'll be navigating via the Terminal which is in the Utilities folder under applications. Once in the UID directory perform a "ls -lh" command to see the files and directories the application is utilizing.

```
mac:8813AE1B-3FC7-4503-99D0-666E974A74D7 travis$ ls -lh
total 0
drwxr-xr-x   4 travis  staff   136B Sep 27 19:08 Documents
drwxr-xr-x   5 travis  staff   170B Jul 11 09:50 Library
drwxr-xr-x  28 travis  staff   952B Sep 25 21:44 iGoat.app
drwxr-xr-x   2 travis  staff    68B Jul 10 21:37 tmp
```

Here we see four directories (Documents, Library, iGoat.app, and tmp), we know they're directories because each line begins with a "d", if it wasn't a directory you would just see a "-" dash. These are the four main directories you'll have for each application and you'll replace iGoat.app with whatever application you're testing (e.g. AppName.app). When it comes to looking for items of interest when security testing one should focus on these four directories, listed below is a chart showing the type of information that can be found in those directories.

![](/assets/iPhoneLocationAppFiles.png)

The contents and location are not set in stone but these locations are the best place to start. So now you know the location of where files may be stored locally on an iOS device but what's the best way to discover the files we're interested in? Let's use what we know in this case, the application tells us that it stores login information into a SQL lite database. Normally you're not going to know that the application uses a SQL lite database to store information unless you're in close communication with the developers of the application. For now we know the app uses a SQL lite database so let's search through these directories looking for that file. We could do this manually but I think dropping to a command prompt is always best. Use the "find" command to locate any SQL lite databases, the command is below.

```bash
mac:8813AE1B-3FC7-4503-99D0-666E974A74D7 travis$ find . -name *.sqlite
./Documents/credentials.sqlite
./iGoat.app/articles.sqlite
mac:8813AE1B-3FC7-4503-99D0-666E974A74D7 travis$
```

From the output of the find command we've located two SQL lite databases for this application, credentials.sqlite and articles.sqlite. Credentials.sqlite is the obvious place to start. There are multiple ways to view that sqlite file but the two I prefer are the [sqlite database browser](http://sourceforge.net/projects/sqlitebrowser/) and the command line utility [sqlite3](http://www.sqlite.org/download.html). I'll be sticking with the sqlite3 command to give you a different perspective as the sqlite database browser is GUI based and easier to use. One thing to note with GUI tool is that it can't browse hidden directories so you'll have to copy and paste sqlite database from the "hidden" Documents directory into a directory that the GUI tool can navigate tool. Next open the sqlite database with the command below.

```bash
travis@mac:Documents$ sqlite3 credentials.sqlite
SQLite version 3.7.14 2012-09-03 15:42:36
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite&gt;
```

You'll be presented with a sqlite prompt. Next issue the ".tables" command into the sqlite prompt.

```bash
sqlite> .tables
creds
```

Here we see a "creds" table, to view the contents of that table use the ".dump" command.

```bash
sqlite&gt; .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE creds (id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT, passwordTEXT);
INSERT INTO "creds" VALUES(1,'super','secret');
DELETE FROM sqlite_sequence;
INSERT INTO "sqlite_sequence" VALUES('creds',1);
COMMIT;
```

Here we see in the first INSERT statement the values of username and password that we typed into the application. So the vulnerability is that credentials to the application are stored in plain text so if someone were to remotely or locally gain access to the iphone then they could glean these credentials. It's always best if possible to store credentials server side and if you must store credentials locally then encrypt that information. This goes for any sensitive information stored locally (text files, config files, plist files, etc). I could extend this example to those types of files but the same concept applies. So if you're curios about one of your iOS apps check out those locations and if develop iOS applications make sure you store local sensitive information in an encrypted format.
