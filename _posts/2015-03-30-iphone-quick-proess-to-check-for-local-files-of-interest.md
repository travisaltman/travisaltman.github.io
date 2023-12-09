---
title: "iphone: quick process to check for local files of interest"
date: "2015-03-30"
layout: post
---

1. Plug iPhone or iPad into Mac
2. User iExplorer or iFunbox to explore file system of apps
3. Export relevant directories to local box (Usually Library and \*.app)
4. Search for files of interest

```bash
find . -name "\*.db"
find . -name "\*.plist"
find . -name "\*.sql\*"
```

Search inside the files for items of interest

```bash
find . -type f -exec grep -l -i "password" {} +
```

iExplorer can open plist in quick view

You can open databases with Sqlite browser

One can read cookie with BinaryCookieReader.py
