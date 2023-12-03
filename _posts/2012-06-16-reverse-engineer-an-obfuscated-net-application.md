---
title: "Reverse engineer an obfuscated .Net application"
date: "2012-06-16"
layout: post
---

Some of the concepts I'll be covering will be new to some people and may be hard to understand but for others who are familiar with this field will find the concepts simple. Hopefully no matter what your comfort level or experience you'll get something out of this.


First I'll give a brief intro to .Net and how it varies from other applications. There two main kinds of applications / programs that you install on your Windows 7 / XP / etc machine, these are either [compiled](http://en.wikipedia.org/wiki/Compiled_language) or [interpreted](http://en.wikipedia.org/wiki/Interpreted_language). Compiled applications can be a [standalone executable](http://stackoverflow.com/questions/2690665/what-are-the-benfits-of-a-standalone-windows-exe-vs-an-installer) but usually you'll have to install it by clicking the usual "Install, Next, Next, Finish" on the other hand an interpreted application doesn't require this. An interpreted application comes as an EXE but an interpreted application does require an "application virtual machine", the interpreted application doesn't run natively on your machine instead the virtual machine takes care of all the execution. In this case the [.Net framework](http://en.wikipedia.org/wiki/.NET_Framework) is the application virtual machine we'll need to have installed. You may have downloaded an EXE in the past and got the message that it requires ".Net framework 3.5" that's because you downloaded an interpreted application. Java is the very similar to the .Net framework in that it employs a virtual machine as well which explains why almost everyone has to have both Java and .Net installed on their windows machine.

Compiled apps are in [assembly machine language](http://en.wikipedia.org/wiki/Assembly_language) and we don't have source code and going from compiled assembly back to original source code is difficult which is why companies like microsoft and apple only give you compiled applications because they want to make it hard for you to get original source code. With interpreted applications it's quite different, for the most part it's easy to decompile interpreted applications back into their original source code. This is true generally because the application virtual machine expects much more information about the application it's about to compile and execute which makes going in reverse easier as well. I may not have explained that well but hopefully this [stackoverflow thread](http://stackoverflow.com/questions/43672/what-types-of-executables-can-be-decompiled) will shed some light.

So by nature of having a .Net application it's easier to get source code from the compiled exe which makes it easier to reverse engineer. Some developers and most users are not aware how easy it is to get source code from applications that run in a virtual machine. Developers that are aware of this sometime utilize obfuscation techniques so that competitors aren't easily able to obtain source code. Obfuscation is simply a way of hiding something you don't want others to know about. Typically it goes "source code --> obfuscation --> hard to understand (garbage)". This is a simple explanation of obfuscation and some techniques are better than others. There are a handful of obfuscation programs on the market that developers can use to hide their code. In the example I'll be showing you the developer used an obfuscation technique but this isn't going to stop us from reversing the program then modifying it to our content.

So the first thing you'll need to do is determine if the exe you have is actually a .Net application and not some other compiled application. There are a couple of tools but the one I like the most is [CFF Explorer](http://www.ntcore.com/exsuite.php). Once installed you can right click on any exe and it'll try and determine what language and compiler was used to make the executable. Let's browse to firefox.exe, right click and choose CFF Explorer.

![](/assets/rightClickFirefox.png)

![](/assets/outputCFFfirefox.png)

From the output of CFF Explorer we can tell that the language Visual C++ was used to create this executable and that it's a PE ([portable executable](http://en.wikipedia.org/wiki/Portable_Executable)) file type. PE's are normally compiled using one of the [C based family languages](http://en.wikipedia.org/wiki/List_of_C-based_programming_languages). This differs from application virtual machine compiled executables such as a .Net application. I've created a small .Net application called GuessPassword.exe that we'll discuss in just a bit but first let's look at the output from CFF Explorer on GuessPassword.exe.

![](/assets/guessPasswordCFFoutput.png)

Here we can confirm that it's a .Net application. So in both cases we have an EXE that we're not completely sure what type of executable it is but with CFF Explorer we can easily find out. So step one was identifying that you have a .Net application, step two will be to decompile the EXE. There are numerous tools out there that can decompile .Net applications. One of the more popular but paid tools is [Reflector](http://www.reflector.net/) but an awesome free open source tool that I'll be going over is [ILSpy](http://wiki.sharpdevelop.net/ILSpy.ashx). So grab the [GuessPassword.exe here](http://travisaltman.com/tools) and open it up in ILSpy. Keep in mind you'll need to either have or install the [.Net framework](http://www.microsoft.com/download/en/details.aspx?id=17718) for all of this to work. Below is a screenshot of what GuessPassword.exe looks like in ILSpy.

![](/assets/guessPassInILSpy.png)

ILSpy will automatically decompile the application, not all at once put as you go down into the tree of the application. So the screenshot above may not mean much to you if it's your first time looking through a decompiled .Net application. There are a couple of things to keep in mind. One only focus on items within the tree of the application you are reviewing, in this case GuessPassword.

![](/assets/ignore.png)

The items that I've marked as ignore are dependencies of GuessPassword.exe that ILSpy pulls in. Secondly you only really need to focus on the "pink bricks" inside of ILSpy. Most of the other components of the tree can be ignored as they're simply tying up loose ends such as defining text boxes the "real" code lies in the pink bricks.

![](/assets/pinkBricks.png)

With this being said go ahead and load GuessPassword.exe into ILSpy and click on the first pink brink named "button1\_Click(object, EventArgs): void", you should see the following.

![](/assets/guessPassFirstPinkBrink.png)

This is a very simple application and from the code above you can probably guess what happens. The application checks to see if the password entered into the text box is "monkey", if it is the message "Correct" is presented if it isn't then the message "Incorrect" will be seen. This is bad practice as you should never hard code passwords into your application but I wanted to show you what it would look like when you're asking for input from the user. Go ahead and run GuessPassword.exe and give it a shot.

![](/assets/guessPassLogin.png)

![](/assets/guessPassCorrect.png)

So this was fairly straight forward and you can see how easy it is to decompile, review code, reverse engineer, and determine ways to subvert the logic of the application. Even though this is considered straight forward the same concepts apply on all applications. At this point we could actually modify this compiled application and change the password "monkey" to anything we like or make it so that we don't even need a password, more on that later.

Now it's time to take our skills to the street. I randomly pulled down an application from download.com called SafeAsHouses. This application is a password safe that keeps all of your passwords in one place, or at least that's the plan. Our goal is to reverse engineer this application so that we can subvert the authentication and glean all passwords. You can grab a copy of SafeAsHouses over at [download.com](http://download.cnet.com/SafeAsHouses-Password-Safe/3000-18501_4-75689621.html?tag=mncol;1) but I've also saved a [copy here](http://travisaltman.com/tools) in case it gets lost in the internets. If you grab it from download.com then the initial password = password and you can change it from there but my copy in the tools directory has the password of "monkey". Next let's throw SafeAsHouses into ILSpy and see what we get.

![](/assets/mainDifferences.png)

As you can see from the screenshot SafeAsHouses immediately shows some signs of obfuscation. In a normal situation the pink bricks will contain the name of the class used, in the case of GuessPassword.exe it's "Form1" but SafeAsHouses is hiding this information. Let's dig further and compare the decompilation of the first pink brick / class for both GuessPassword and SafeAsHouses, first GuessPassword.

![](/assets/firstClassGuess.png)

Now SafeAsHouses.

![](/assets/firstClassSafe1.png)

Once again we see signs of obfuscation. Anywhere you see a gray box is a sign of obfuscation and this obfuscation technique is essentially hiding names within the code such as variable names. This won't stop us but it does make the reversing process a little bit more difficult. So now you've successfully decompiled the application and see the signs of obfuscation, at this point you could start digging through code but it's always to run the application to get a better understanding of what the application is doing.

![](/assets/loginSafeAsHouses.png)

![](/assets/mainWindowSafeAsHouses.png)

So soon as you launch the application we see that it's asking for a password, after typing in the correct password we'll have access to all of our usernames and passwords. If we enter the incorrect password the application will show a message box saying "Password incorrect".

![](/assets/safeHousePasswordIncorrect.png)

So we're trying to figure out how to get around the whole password authentication, that being said there are several areas we can focus on. The first is the fact that the application asks for input before allowing us full access into the application. The second is that the application will close if the input is incorrect. What [strings](http://en.wikipedia.org/wiki/String_%28computer_science%29) can be seen during this process? Another thing to consider is the fact that the application presents you with one window for authentication then shows you a second window where your passwords are stored.  For each consideration you should ask yourself what would the code look like in a .Net application.

In the case of input it's more than likely form input. In my simple GuessPassword.exe the code that takes is "this.textBox1.Text" where I provided the name textBox1 which is actually the default variable name give inside of Visual Studio. So in the application you're trying to reverse you would need to look for a similar construct, this can be done manually or you can search through the decompiled code. Keep in mind when searching or viewing manually an obfuscated application you probably won't see something similar to "this.Variable.Text", at the very least an obfuscation tool will mangle "Variable" so keeping your eyes open for "this.something.Text" maybe your next best step.

So the application closes when an incorrect password is typed in, what does the code look like to close an application in .Net? For the most part the application will be closed with an [Application.Exit](http://msdn.microsoft.com/en-us/library/ms157894.aspx) or [Environment.Exit](http://msdn.microsoft.com/en-us/library/system.environment.exit%28v=vs.80%29.aspx) so when you go to reverse the application those are the constructs you'll need to keep in mind for when you search or manually review the decompiled code.

I mentioned "strings" earlier, one thing we notice when the application closes is "Password incorrect" and the title of the window is "ACCESS DENIED". These strings have to be somewhere in the source code, they might be obfuscated but it's definitely a good idea to search for them within the source code. If they aren't in the source code then what other constructs generate these messages? If you decompile my GuessPassword.exe you'll notice that my incorrect password message is "Incorrect" and that the title of the window is "Incorrect" but these are generated by [MessageBox.Show](http://msdn.microsoft.com/en-us/library/0x49kd7z.aspx) so if the obfuscation tool hides the phrase "Incorrect" it may not hide the function "MessageBox.Show" so it's just another thing to consider when trying to reverse a .Net application.

Last but not least is the fact that the application goes from one small window that takes your password to a second window that has all your passwords, how does the application do this and what does the code look like when this is performed in a .Net application? This can be handled in a number of different ways. Probably the most popular method is using the "window.Show" and "window.Hide" functions. Another option is using form opacity, form opacity is typically used to make a window more transparent but you could make the window completely transparent with this option as well. So when searching through code it's a good idea to keep both of these functions in mind.

Now that I've laid out some things to consider now it's time to put this to practice. For each consideration I've laid out you can manually look through the decompiled code for these constructs but ideally we'll want to search for these constructs. At the time of this writing I've found a couple of ways to accomplish this. One is to decompile all the code of the application into a text file then search that text file with your favorite text editor. For this I prefer ILSpy to decompile into one text file then I can search inside that text file for any construct I like. First select the program you want to save into a file.

![](/assets/selectProgram.png)

Next choose File > Save Code

![](/assets/saveCode.png)

Then save to a "single file", you can save to a project but for searching through code I prefer one flat file.

![](/assets/singleFile.png)

Now that you have that single file you can open up your favorite text editor and search for any construct or item of interest. Go ahead and search for MessageBox.Show and see what hits you get.

![](/assets/textSearch.png)

So we see present a message box but these message boxes are all throughout the application so it's hard to say that this is the functionality we're looking for. Another thing to note is that the obfuscation is hiding what the message is with large negative numbers. Having all the code in one text file is helpful but it doesn't put much context around what we're trying to go after. Most developers don't code inside one giant text file they usually develop inside an IDE such as Visual Studio which breaks up the code to help with contextual understanding. That being said whenever I'm looking at an application that can be decompiled I always save that decompiled code into text files so that I can search it at anytime for items of interest, IDE's are nice but plain text rules.

Another great option to search for constructs is using the plugin [CodeSearch](http://reflectoraddins.codeplex.com/wikipage?title=CodeSearch&referringTitle=Home) for the .Net Reflector tool. What's great about this tool is that if you're already working inside Reflector or if Reflector is your favorite tool for assessing .Net applications then you have everything in one place. Let's do the same messagebox.show search inside of the CodeSearch plugin. One thing you'll need to keep in mind with the CodeSearch plugin is that it will only search through what you've clicked on in the left hand column. So if you want to search the entire tree of SafeAsHouses you'll need to make sure you're on the root of that application. Below you'll see that Reflector tags SafeAsHouses as PasswordSafe, so first I clicked on PasswordSafe then performed my search in CodeSearch.

![](/assets/codeSearch1.png)

The nice thing about CodeSearch is that it will break down the results into the path were the results can be found. CodeSearch is case sensitive so that's another thing you'll need to keep in mind when performing searches. As you can see from the screenshot above we have quite a number of hits from "MessageBox.Show" this is probably because it's a popular function to use in .Net applications. You can click through each result and it will take you to the location of where it found your search term, from there you'll have to determine if that location has the functionality you're looking for. Usually it's best to search for all the relative constructs to get the best idea of where the functionality you're looking for resides. For example you should also search for "this.\*.Text" looking for form input, "Environment.Exit" when the application closes, "Opacity" "\*.Hide" for hiding windows / forms, and "\*.Show" for showing windows, etc. So it's a good idea to search for all these terms as it relates to SafeAsHouses to find the code that actually does the authentication piece. In this case the search term "Opacity" does a really good job of narrowing down the location of the code we're interested in.

![](/assets/searchResultOpacity.png)

So looking at the decompiled code in the top right column it appears that this is the authentication functionality we've been looking for in the SafeAsHouses application. Congratulations we found the functionality we've been searching for. It may not seem that obvious but let's take a closer look, I prefer to use ILSpy in this case because it does a better job of showing the original code.

![](/assets/ILSpyOutput.png)

On line six the developer is assigning the variable to whatever you type into the password field, he's then comparing that to the obfuscated value we see on line seven. If the password is correct you then enter the if statement, this is crucial you have to enter the if statement in order for the application to open / show you the passwords. I don't completely understand the role of the opacity code but the code "base.Hide();" on line 17 hides the password window and the code "SO.Show" (where SO is the obfuscation) will show you the second window which contains all your passwords. Line 22 is the pop up message that tells you that your password is incorrect and line 23 closes the application because you entered the wrong password. So after looking at all the functionality in this section we can determine that this is definitely the section we need to focus on. Now we need to figure out how we can modify the code to bypass the authentication. The idea here is that someone has downloaded this application to their machine and use this application to store all their super secret passwords, what we want is to do is figure out all their passwords by modifying / patching the application. With a .Net application it's fairly easy to modify to get your desired goal. There are a couple of tools that will allow us to modify a .Net application, [GrayWolf](http://digitalbodyguard.com/GrayWolf.html) and the Reflector plugin [Reflexil](http://reflexil.net/). Both are great tools and must haves in your .Net reversing toolbox but for this demonstration I'll stick to reflexil. I'm not going to discuss installing reflexil so please refer to the documentation on how to get that setup. Once the plugin is installed simply go to Tools > Reflexil inside of Reflector, you should see something similar to the screen shot below.

![](/assets/reflexil.png)

So the upper half of this screen shot is the authentication code we've identified for the SafeAsHouses application and the bottom half is reflexil. What reflexil shows you is the CIL ([common intermediate language](http://en.wikipedia.org/wiki/Common_Intermediate_Language)) often called IL for short. You can read through the wiki article that I've linked to but basically it's the lower level language used by the .Net virtual machine. For example, you write your code in C Sharp, that code then gets converted to CIL. The CIL is ran through the virtual machine which got installed when you installed the .Net framework on your computer. It's not important that you understand all the inner workings of the .Net framework all you really need to know is that if we want to modify a .Net application then having a general knowledge of CIL will come in handy. I wouldn't try to completely understand CIL out of the box because it shouldn't be too difficult to follow the CIL instructions and map them to the decompiled code. For instance we can compare lines 01, 02, 03 in the CIL to the decompiled code and piece together that System.Window.Forms.TextBox plus the other instructions are getting input from the user via a text box inside a windows form. This is also consistent with how the application behaves.

The next step is to use reflexil to modify the CIL then save those modifications. What we want to do is eliminate the need to provide a valid password, this can be accomplished a number of ways. You'll notice in the CIL on line 07 there is a CIL instruction, "op\_Equality", that compares the user supplied password to the actual password associated with the application. You'll notice on line 08 the OpCode instruction is "[brfalse](http://msdn.microsoft.com/en-us/library/system.reflection.emit.opcodes.brfalse.aspx)", this essentially means that if the passwords don't match (false) then go into that if clause which will pop up a message and the application will exit. You'll also notice the operand for the brfalse opcode is "-> (36)", this means branch to target on line 36 which jumps into the message box functionality. To get around providing the correct password we can simply change the condition from false to true that way if we provide the incorrect password it will open the application to whereas providing the correct password will cause the application to close. To change the CIL right click on brfalse and select edit.

![](/assets/rightClickReflexil.png)

Now change the opcode from brfalse to brtrue then click update.

![](/assets/reflexilWindow.png)

Once you've modified the CIL instruction you'll need to save your changes. To do this right click on the root of your application, then go to reflexil, then save as.

![](/assets/saveModified.png)

I typically stick with the \*.Patched naming that way I don't modify the original executable. After you've saved SaveAsHouses.Patched go ahead and double click on the application. It should work out that if you supply the incorrect password you'll be authenticated but if you supply the correct password the application will close.

Congratulations, that's it you've successfully reversed an obfuscated .Net application and modified the executable to subvert authentication. At this point you've accomplished your mission but you could have gone another route. Instead of changing the brfalse to brtrue we could have deleted all those instructions that way it doesn't even check for the password. The code block below shows what can be deleted to achieve this functionality.

![](/assets/deleteCode.png)

In order to delete this code within the application we'll need to delete lines 1-28 within reflexil, you can do the usual select line one then while holding shift select line 28. After that right click and select delete then save the changes. Open your patched version of SafeAsHouses and type whatever you like into the password field, this can be the correct password or not it doesn't matter. Congratulations you've accomplished another method to subvert functionality of a standalone .Net application. Just to prove we've removed that functionality of the application open up the patched version of SafeAsHouses in either Reflector or ILSpy, below is my screenshot that shows the removed code.

![](/assets/patchedCode.png)

So there you have it, hopefully this explanation was helpful. I've been kind of hiding a secret Relfexil has an option called "Obfuscator search" which will automatically try and deobfuscate code. This tool actually works really well and it will deobfuscate SafeAsHouses for you automatically. Even though Reflexil can remove obfuscation on some applications it can't do them all and even it could the concepts that I've covered still apply. So even if you have a deobfuscated application you'll still need to identify certain constructs within the code to find the functionality you're looking for.

I'm not here to bash on .Net it's actually a great platform to quickly stand up applications but people need to be aware that it's not incredibly difficult to reverse the application, tear it apart, and inject / subvert functionality. That's all I got, if you see where I've made a mistake or left out some important information please let me know. Happy .Net hunting.
