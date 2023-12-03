---
title: "password dictionary generator"
date: "2010-03-06"
layout: post
mathjax: true
---

I had the need to generate a password dictionary that would cover every possible combination for a defined character set.  I first learned to program in Python so I was going to start there first.  Before writing the program I decided to Google and see if anyone else had tackled this problem via Python, turned out they had.  [Siph0n posted his Python code](http://forums.remote-exploit.org/programming/14204-another-password-wordlist-generator-python.html) to create a password dictionary over at the BackTrack forums.  I wanted to post it here as a mirror and to discuss the implications of creating a password dictionary with every possible combination.  Below is the Python code.


```python 
f=open('wordlist', 'w')

def xselections(items, n): 
  if n==0: yield \[\] 
  else: 
    for i in xrange(len(items)):
      for ss in xselections(items, n-1):
        yield \[items\[i\]\]+ss

\# Numbers = 48 - 57 
# Capital = 65 - 90 
# Lower = 97 - 122
numb = range(48,58)
cap = range(65,91)
low = range(97,123)
choice = 0
while int(choice) not in range(1,8):
  choice = raw\_input('''
  1) Numbers
  2) Capital Letters
  3) Lowercase Letters
  4) Numbers + Capital Letters
  5) Numbers + Lowercase Letters
  6) Numbers + Capital Letters + Lowercase Letters
  7) Capital Letters + Lowercase Letters : ''')

choice = int(choice)
poss = \[\]
if choice == 1:
  poss += numb
elif choice == 2:
  poss += cap
elif choice == 3:
  poss += low
elif choice == 4:
  poss += numb poss += cap
elif choice == 5:
  poss += numb poss += low
elif choice == 6:
  poss += numb poss += cap poss += low
elif choice == 7:
  poss += cap poss += low

bigList = \[\]
for i in poss:
  bigList.append(str(chr(i)))

MIN = raw\_input("What is the min size of the word? ")
MIN = int(MIN) MAX = raw\_input("What is the max size of the word? ")
MAX = int(MAX)
for i in range(MIN,MAX+1):
  for s in xselections(bigList,i): f.write(''.join(s) + '\\n')
```

If you're familiar with programming and Python in particular then you could just grab the code and roll but I really wanted to discuss the usefulness of an application like this.  First I will discuss the basics of how to get this program up and running but will eventually jump into other implications such as time, storage, and usefulness of a password dictionary.

How to install and use the program

1. You must have Python installed.  If you're running Linux (you should be) then it's probably already installed.  If you're running then Windows then you will have to [download Python](http://www.python.org/download/).
2. Now that you have Python installed simply copy and paste the code above into a text file and name it passwordDictionaryGenerator.py.  The .py extension is needed because that's how Python recognizes code that it's suppose to execute.
3. Modify appropriate variables within the program.  The only variables you may want to modify are numb, cap, and low.  These variables contain the ASCII equivalent ranges for the letters and numbers you will be using to generate your dictionary.  You may want to modify these variables so that your dictionary does not contain a-z but only a-k, I'll leave that up to you.
4. Now to run the program simply type \[cc lang=python\]python passwordDictionaryGenerator.py\[/cc\]You will have to answer the questions about which character set you want to use and how long / short your password dictionary is going to be.  Once you answer the questions it may seem like the program isn't doing anything but it is, it will spit you back to the command line once the program has completed.  The output will be a file called wordlist.

So now you have this cool program that can generate a password dictionary for you, how big (size MB, GB, TB, etc) will this dictionary be?  How long will it take to generate this dictionary?  Let's tackle the size question first as it will help us calculate the time as well.  The key to calculating the size is a math term called permutations.  [Permutations](http://www.aaaknow.com/sta-permu.htm) is a simple equation to determine the number of words for that particular character set and length of word.  The basic equation is below.

$$ n^r $$

n = total character set (e.g.  a-z + A-Z + 0-9 = 62)

r = length of the word

Now you'll have to calculate nr for each length to get every possible combination.  So for a 6 digit long password your equation will look like the following.

n6 + n5 + n4 + n3 + n2 + n1 = every possible combination

Let's try an example where our character set is a-z (n = 26) and our password is no longer than 6 (r = 1-6) digits, how many words will be in our dictionary?

266 + 265 + 264 + 263 + 262 + 261 = 321,272,406 = total # of words

So now we understand how to calculate the total number of words in our dictionary.  How does that relate to the size?  Well for the most part if the length of the password is x then the size in bytes will be x + 1 for that particular line.  Then all we have to do is multiply each nr times the size of that particular line to get the size for that particular length.  That may have just sound really confusing so hopefully the following graph clears that up some.

![](/assets/possibleCombinationChart.png)

I went ahead and generated this dictionary, it took about 30 minutes.  Turns out the size matched my calculations.

![](/assets/wordlistSize.png)

So now you have the basic formula for calculating the size of your desired dictionary.  Let's take a look at a larger example just to cure our curiosity.  Let's assume the following parameters.

- character set = a-z, A-Z, & 0-9
- password length = 1-8
- n = 62
- r = 1 - 8

With these parameters the size of our dictionary jumps to 1,800 terabytes or 1.8 petabytes. Take a look at the chart below.

![](/assets/possibleCombinationChart2.png)

You can see how quickly the size jumps up. I don't know about you but I don't have a two petabyte drive lying around. Generating this dictionary is just infeasible. I did calculate the time it would probably take to generate this dictionary, it came out to be about 11 days. So the time to create such a dictionary is nothing compared to the storage required to house it. Not only that I don't know to many applications that can handle a large dictionary as input, so that's another factor you'll have to keep in mind when generating your dictionary.

Calculating the time it takes to generate these dictionaries I'll leave up to you.  The basic idea is that you can run the python program for a particular length password for a set amount of time and then extrapolate form there.  For the most part time isn't really a factor but storage is. The concepts I've talked about here are nothing new. The idea of generating a password came to me and my coworkers as we were thinking of ways to test a WPA wireless infrastructure. Attacking WPA can be done offline so we were thinking of generating a dictionary to accomplish this. Hours later we soon realized the difficulty with generating such a large dictionary. This was actually good news because it meant that an attacker would have an extremely difficult time attacking a WPA access point with a complex password. [Renderman and the Church of Wifi](http://www.renderlab.net/projects/WPA-tables/) have thought about this problem way before I did and came up with some rainbow tables to help test the strength of your WPA access point. You can't really create a dictionary with every single combination for a lengthy password, your best bet is to create a dictionary with the most "common" passwords, which is no easy task either.

The moral of the story is to use lengthy complex passwords with a high character set, but you knew that already. So I just suggested that this program is somewhat useless, well it is but it isn't. You can use this program to generate a small dictionary but a large dictionary (greater than a couple of terabytes) is probably out of the question. So use this program and let me know what your results are, I'm always interested in your feedback. Happy cracking.
