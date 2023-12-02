---
title: "Search windows open shares with python"
date: "2011-09-02"
categories: 
  - "programming"
  - "windows"
---

It's rare during a penetration test that I actually exploit a vulnerability to gain more information. Newcomers to my filed will often use the term "network security". I don't care about the network, have the network for all I care. What I'm more concerned about is the information inside the network. The better way to describe it is "information security". Performing penetration tests one has to keep that in mind, yea it's fun to exploit some user that's running an old version of war-ftp but if that user doesn't yield sensitive information then who cares to some extent.

I often see that professional penetration testers will highlight an open windows share that can be read or written to by everyone. They will often highlight other shares that are accessible by a large group such as Authenticated users. I don't want to scoff at these types of open shares as they should be investigated by the business owner that created the open shares. The main thing to consider is what information lies within those open shares. Open shares are usually created for a reason, so that users easily share information. This is not bad unless the information in those shares is secret / classified material. To check for this possible sensitive information one would have to search all the files and folders in that share. Now you can use the cute little dog search feature inside of windows explorer to look for this information but using that your hands are somewhat tied. The search feature inside windows explorer actually does a nice job but if you wanted to automate the process to look at multiple shares and search for multiple terms then you're out of luck. Because of this I wanted to script something that would automate the process. Powershell could have been an option but because I'm already familiar with python I stuck to what I know. This means that in order to run the script you'll have to have python installed on windows. I could have written the script to work in Linux but that would have meant using cifs to map drives which seemed like more of a headache then just using python on windows.

You'll need to open up a windows command prompt to run the script and it's a good idead to [add Python to the windows path](http://showmedo.com/videotutorials/video?name=960000&fromSeriesID=96). So the script takes two arguments. The first argument is the file containing all the shares that you want to search. The second argument is the file that contains all the terms you want to search for. So to run the script you would issue a command similar to below, where searchShares.py is the name of the python script.

```
python.exe  searchShares.py  shares.txt  searchTerms.txt\
```

Your shares.txt file should look similar to below.

```
\\one\two
\\three\four\\five
\\six\seven\eight\nine
```

Your searchTerms.txt file should look similar to below.

```
secret
password
username
```

In the example above the term "secret" will be recursively searched in all three shares. Then "password" will be recursively searched in all three shares, then so on and so on. The script will output any file, file name, or folder name that matches any of the search terms. Currently the script will read each file in [binary format](http://en.wikipedia.org/wiki/Binary_file) which means if it comes across a word document file (such as document.doc) it doesn't open / read the file like microsoft word would. The current script reads each line of the binary file looking for your search term. Reading a text file as binary seems to work fine but reading in microsoft office documents as binary have different results. One thing I've noticed in my testing is that generally speaking it does just fine searching through a \*.doc file but has trouble searching through a \*.docx file. Binary searching is not ideal but it's my current solution. Python has the capability to open microsoft office documents in a more native format but for my first go round I haven't implemented that solution.

Once you run the script you will see output similar to below.

```bat
C:\\temp>python searchShares.py shares.txt searchTerms.txt
Walking directory \\\\192.168.99.184\\test
Found \\\\192.168.99.184\\testtest.txt Found \\\\192.168.99.184\\testTravisAltmanResume.doc Found \\\\192.168.99.184\\test\\onewordDoc1.docx Found \\\\192.168.99.184\\test\\one\\twopasswords.txt Found \\\\192.168.99.184\\test\\one\\two\\threewordDoc2.docx Searching file \\\\192.168.99.184\\test\\test.txt for term secret
Searching file \\\\192.168.99.184\\test\\TravisAltmanResume.doc for term secret
Searching file \\\\192.168.99.184\\test\\one\\wordDoc1.docx for term secret
Searching file \\\\192.168.99.184\\test\\one\\two\\passwords.txt for term secret
Searching file \\\\192.168.99.184\\test\\one\\two\\three\\wordDoc2.docx for term secret
Searching file \\\\192.168.99.184\\test\\test.txt for term password
Searching file \\\\192.168.99.184\\test\\TravisAltmanResume.doc for term password
Searching file \\\\192.168.99.184\\test\\one\\wordDoc1.docx for term password
Searching file \\\\192.168.99.184\\test\\one\\two\\passwords.txt for term password
Searching file \\\\192.168.99.184\\test\\one\\two\\three\\wordDoc2.docx for term password
Searching file \\\\192.168.99.184\\test\\test.txt for term username Searching file \\\\192.168.99.184\\test\\TravisAltmanResume.doc for term username Searching file \\\\192.168.99.184\\test\\one\\wordDoc1.docx for term username Searching file \\\\192.168.99.184\\test\\one\\two\\passwords.txt for term username Searching file \\\\192.168.99.184\\test\\one\\two\\three\\wordDoc2.docx for term username
```

This output on the command prompt is to given as a verbose message so that you know what's going on with the script. The output on the command prompt will not tell you if it found a search term. The results of your searching is placed in a text file called output.txt located in the current directory. The content of output.txt should look similar to the following.

```bat
\=== Directories or file names matching search criteria ===
\\\\192.168.99.184\\test\\one\\two\\passwords.txt
\=== Files matching search criteria ===
found secret in file \\\\192.168.99.184\\test\\one\\two\\passwords.txt found password in file \\\\192.168.99.184\\test\\one\\two\\passwords.txt
```

So you can see that it matches the file name as well as the contents of the file. One thing to keep in mind is that this script can take a while to run. There two factors that control how fast it runs, 1) Speed of the network and 2) Size (GB, MB, etc) of the share. It works best when your network is local and not in another city. The biggest factor is going to be the size of the share. Running this script on a major file sahre that is say 800 GB in size will take a very long time. Keep in mind you can specify specific directories, so instead of searching in the root share such as \\\\share\\one maybe it's a better idea to searh in \\\\share\\one\\two\\three. So keep these factors in mind when running the script. Below is the script, simply cut and paste into your text editor of choice and save as searchShares.py

```python
import os
import sys
import re

output = open('output.txt', 'a')
output.write('\\n')
fileList = \[\]
shareList = open(sys.argv\[1\])
eachShare = shareList.readlines();
for shares in eachShare:
    path = shares.rstrip('\\r\\n')
    print '\\nWalking directory ' + path + '\\n'
    for root, subFolders, files in os.walk(path):
        #print 'Indexing ' + root + '\\n'
        for file in files:
            fileList.append(os.path.join(root,file))
            print 'Found ' + root + file
keywords = open(sys.argv\[2\])
searchTerm = keywords.readlines();
output.write('=== Directories or file names matching search criteria ===\\n')
for term in searchTerm:
    strip = term.rstrip('\\r\\n')
    if any(strip in s for s in fileList):
        matching = \[s for s in fileList if strip in s\]
        for item in matching:
            output.write('\\n' + item)
output.write('\\n\\n=== Files matching search criteria ===\\n\\n')
for term in searchTerm:
    strip = term.strip('\\r\\n')
    for item in fileList:
        print 'Searching file ' + item + ' for term ' + term
        searchFile = open(item, 'rb')
        for line in searchFile:
            if re.search(strip, line, re.IGNORECASE):
                output.write('found ' + strip + ' in file ' + item + '\\n')
                break
    searchFile.close()
output.close()
```

Let me know if this works / doesn't work and also let me know if you have any suggestions on how to make it better. One thing I might do in the future is to limit the types of files it searches to say only .txt, .doc, .xls, etc. Happy hunting for information on shares.
