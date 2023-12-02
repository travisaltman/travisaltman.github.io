---
title: "Search an IP range via the command line"
date: "2009-09-05"
post: layout
---

So how do you manipulate a list of IP's via the command line? Well there are several ways to go about this but I'll present the way I went about it.

In my scenario I had a range of IP's that I needed to extract/exclude out of a list of IP's. This task needed to be done on a Windoze machine, I do most of my scripting on a Linux box, so I was trying to rely on the findstr command. Trying to use the [findstr command](http://ss64.com/nt/findstr.html) to search, extract, or manipulate a list of IP's will make you crazy. Now I'm sure there's way smarter people out there that can craft a simple one line findstr command to hack and slash on an IP list but I'm not one of those people. I also tried to utilize some regular expression magic to manipulate an IP range. Google has this [regular expression generator](http://www.google.com/support/analytics/bin/answer.py?hl=en&answer=55572) specifically for IP ranges, which seems neat at first but I couldn't get it to work within findstr.


After no luck with findstr I was gonna turn to my old friend grep. Now for those that don't know grep is a pattern / regular expression matching command within Linux. Grep has the ability to search for patterns within directories and files for a specific string (e.g. IP addresses). There is a [grep Windows executable](http://www.thedance.net/~win95/grep.exe) with basically the same functionality but it couldn't handle Google's regular expression either. After burning through two different programs to perform this task I was almost at a lost. My coworker reminded me of [awk](http://www.amazon.com/Effective-awk-Programming-Arnold-Robbins/dp/0596000707/ref=sr_1_2?ie=UTF8&s=books&qid=1252164251&sr=8-2), how could I forget. Awk is a native program within Linux but you can download an exe version of the program. There are different flavors of awk (gawk and mawk) and different programmers that try and port over awk. I tried some awk.exe's and some gawk.exe's but I had the best success with mawk.exe, you can grab [mawk.exe here](http://travisaltman.com/tools/mawk.exe). So enough yip yapping let's walk through the solution. Below is a sample list of IP's that we'll hack and slash on, let's assume these IP's are in a file called IPlist.txt.

```
192.168.0.1
192.168.0.2
192.168.0.3
192.168.0.4
192.168.0.5
192.168.0.6
192.168.0.7
192.168.0.8
192.168.0.9
192.168.0.10
192.168.0.11
192.168.0.12
192.168.0.13
192.168.0.14
192.168.0.15
192.168.0.16
192.168.0.17
192.168.0.18
192.168.0.19
192.168.0.20
192.168.5.1
192.168.5.2
192.168.5.3
192.168.5.4
192.168.5.5
192.168.5.6
192.168.5.7
192.168.5.8
192.168.5.9
192.168.5.10
192.168.5.11
192.168.5.12
192.168.5.13
192.168.5.14
192.168.5.15
192.168.5.16
192.168.5.17
192.168.5.18
192.168.5.19
192.168.5.20`
```
So let's say we wanted to extract or exclude the range 192.168.0.5-192.168.0.15, you would use the mawk command below.

```
mawk "BEGIN {FS='.'}; $3<0 || $3>0 || ($3==0 &&($4<5 || $4>15)) {print $0}" IPlist.txt`
```

Let me explain the command above. BEGIN simply processes the text before mawk starts munching. FS stands for field separator, here we are telling mawk that our filed separator is period (surrounded by single quotes). The $3 is basically a variable calling the 3rd field, in our case it's the third number in our IP address. The || means "or". The == is to determine is something is equivalent. The && is "and". The $4 is the 4th number in our IP address because it's the 4th field. So the command reads like this: separator is a period, we want the 3rd number to be less than zero or greater than zero or equal to 3 and we want the 4th number to be less than 5 or greater than 15. The $0 represents the entire line so the print statement is just printing out the entire line that matches our criteria. Let's look at a similar example, say we want to extract 192.168.5.10-18.

```
mawk "BEGIN {FS='.'}; $3<5 || $3>5 || ($3==5 &&($4<10 || $4>18)) {print $0}" IPlist.txt`
```

I'm sure there are probably other ways to go about performing the same task but this one works for me. Now feel free to go ahead and [mawk it out](http://www.youtube.com/watch?v=pxjZM-d_ShI).
