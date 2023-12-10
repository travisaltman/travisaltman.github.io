---
title: "Encode your SQL injection attacks"
date: "2015-10-30"
layout: post
---

Encoding SQL injection attacks is nothing new and automated tools like SQLmap will more than likely find flaws via this method.  That being said I was combing through some of my old docs and found what I think is a decent explanation how this type of attack leads to SQL injection.


I'll be demonstrating this on an app called GetBoo that is installed on the [OWASP broken web app project](https://www.owasp.org/index.php/OWASP_Broken_Web_Applications_Project) so feel free to download the virtual image they provide and begin playing with GetBoo and other similar broken applications.  Inside GetBoo we look for some SQL injection, unauthenticated vulnerabilities are always best if you can find them. Going to the home page of GetBoo we see there are comments posted to a bookmark. Hovering over the comment link and also clicking on the comment link takes us to http://172.16.114.218/getboo/comment.php?bID=2, keep in mind your IP address will be different. In Burp let’s send this request to repeater so that we can analyze if the bID parameter is vulnerable to SQL injection. First thing we’ll want to do is throw in the classic single quote, doing this responds back with a 500 server error.

![getboo1](/assets/getboo1.png)

This does not mean the application is vulnerable to SQL injection but it is an indicator that the application can’t handle the single quote without failure. Let’s look at a request that handles the single quote just fine, at the home page you can click on “Popular Tags”, after that click on sort by “Popularity”. Send that request to repeater in Burp and instead of “popularity” for the sort parameter place a single quote in for the value of sort. You will notice that the application handles the request just fine with a 200 OK status.

![getboo2](/assets/getboo2.png)

The other classic SQL injection string ‘ or 1=1 also gives us a 500 error, most of those classic ‘ or 1=1 attacks are followed by a – which is a comment in Microsoft SQL server. In this example we’re running MySQL so if we wanted to follow it up with a commend it would look like ‘ or 1=1#.

![getboo3](/assets/getboo3.png)

Still getting a 500 error. Let’s try another technique which is to finish out or complete the request with a single or double quote after a valid parameter value.

![getboo4](/assets/getboo4.png)

So finishing off the request with a single quote and appending the and 1=1 still results in the same error. You can try the same attack without the single quote to see the affect as well.

![getboo5](/assets/getboo5.png)

Just because we get a 200 OK doesn’t mean that our SQL injection was successful, if you put other meaningless data besides 1=1 you may not notice a change.

![getboo6](/assets/getboo6.png)

![getboo7](/assets/getboo7.png)

Here we see it doesn’t matter what we append after a valid value for the bID parameter because the application is simply ignoring anything after, so this isn’t going to lead us down the path to SQL injection at least for this particular parameter so somehow we need to focus on finishing off the query with either the single quote or another technique.

With GetBoo installed locally you can gain access to the Mysql database logs which is extremely helpful when trying to debug a successful SQL injection attack of course in the real world this is unlikely to happen but if you want to perform better testing on your application then giving application testers access will go a long way.

Let’s go back to our single quote attack and see the output from the database query.

```
140805 3:31:05 750 Connect getboo@localhost on getboo
750 Query SELECT config_name, config_value FROM configs
750 Query select title from favourites where id='2''
750 Quit
```

Here we see that it is keeping the request as we sent it. In this case if the attack string bID=2’ and 1=1 is sent you get the same output from the Mysql database logs, this means the application is discarding anything after the single quote. One way of getting around that may be to encode your attack string. This time we will URL encode our attack string, so instead of the request being bID=2’ and 1=1 it will now be bID=2 %27%20%61%6e%64%20%31%3d%31. Once we run that the following shows up in the Mysql logs.

```
140805 3:43:25 755 Connect getboo@localhost on getboo
755 Query SELECT config_name, config_value FROM configs
755 Query select title from favourites where id='2' and 1=1'
755 Quit
```

Encoding the attack made all the difference although we still get a 500 error when making the request so we’ll need keep hammering away.

![getboo8](/assets/getboo8.png)

Not all is lost though because notice in the Mysql output that a single quote is appended on the 1=1 even though we didn’t specify a single quote at the end of the attack. So instead of using numeric values we need to use strings and quote those strings but leave off the final quote as the application will put that in place for us. So in this case we can use the attack string bID=2’ and ‘blah’=’blah and the URL encoded value is bID= %32%27%20%61%6e%64%20%27%62%6c%61%68%27%3d%27%62%6c%61%68.

![getboo9](/assets/getboo9.png)

Great success we were able to get a meaningful 200 response to our attacks. In the output from Mysql we can see that it properly parsed our attack as a valid SQL statement.

```
140805 4:03:19 765 Connect getboo@localhost on getboo
765 Query SELECT config_name, config_value FROM configs
765 Query select title from favourites where id='2' and 'blah'='blah'
765 Query select b_id from tags_books where b_id='2' and 'blah'='blah'
765 Query select title from favourites where id='2' and 'blah'='blah'
765 Query select b.ADD_DATE AS formatted_time, id, title, url, description, name from favourites b, tags_added ta where (b.id = ta.b_id and b.id = '2' and 'blah'='blah')
765 Query select title from tags t, tags_books tb where (t.id=tb.t_id and tb.b_id = '2')
765 Query select title from comments where bid='2'
765 Query select title, comment, author, date from comments where bid='2' and 'blah'='blah'
765 Query SELECT t.title, count(tb.t_id) as amount from tags_books tb, tags t, favourites f where t.id = tb.t_id and tb.b_id = f.id and f.name = 'user' group by tb.t_id, t.title order by amount desc LIMIT 0, 35
765 Quit
```

Now that we confirmed that we’re actually making proper database requests we can begin to pilfer the contents of the database.
