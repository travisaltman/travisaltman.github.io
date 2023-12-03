---
title: "metasploit set rhosts file"
date: "2015-01-31"
layout: post
---

Just a quick tip I don't see documented a bunch of places, when you want to feed metasploit a list of targets in a file you need to use the following syntax.

```bash
set rhosts file:/path/to/file\
```

This file will need to be values separated by a new line. Below is a screenshot for context.

![](assets/metasploit-set-rhosts-file.jpg)
