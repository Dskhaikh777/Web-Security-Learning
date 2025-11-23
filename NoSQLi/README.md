# NoSQL Injection

## Table of Contents
- [Labs](#labs)
  - [Lab 1: Detecting NoSQL injection](#lab-1-detecting-nosql-injection)
  - [Lab 2: Exploiting NoSQL operator injection to bypass authentication](#lab-2-exploiting-nosql-operator-injection-to-bypass-authentication)
  - [Lab 3: Exploiting NoSQL injection to extract data](#lab-3-exploiting-nosql-injection-to-extract-data)
  - [Lab 4: Exploiting NoSQL operator injection to extract unknown fields](#lab-4-exploiting-nosql-operator-injection-to-extract-unknown-fields)

# Labs:-

## **Lab 1: Detecting NoSQL injection**

In this lab, we just have to detect the vulnerability.

First, I captured the product category request using Burp Repeater. After that, I inserted a **single quote** to check if an error would occur. I received an error 500, which also shows that the website is using MongoDB.

then I try to check if the application is accepting boolean conditions and the response is changing or not.

For that, I used:

`' && 0 && 'abc` —> It does not show anything.

`' && 1 && 'abc` —> It show the product of the respective chategory.

then I try to retrieve all products from all chategories:

`'|| 1 ||'` —> return all the products.

---

## **Lab 2: Exploiting NoSQL operator injection to bypass authentication**

In this lab, we have to exploit NoSQL operator injection and log in as an administrator.

First, I capture **`/login`** request  and in the JSON username parameter is present, and I check that parameter by  changing its value to **`{"$ne":""}`**

and it is working fine.

same, I check for the password parameter with username wiener. I log in as wiener, which means both parameters are vulnerable to NoSQL Injection.

Now, I know the Wiener user exists, so I changed both parameter values to **`{"$ne":""}`**

It returns the error 

**`Internal Server Error`**

`Query returned an unexpected number of records`

this means more than one user exists.

Finally, I change username parameter to **`{"$regex":"admin.*"}`  (It will match the username start with admin….)**

and password to **`{"$ne":""}`**

I successfully log in as an Administrator.

---

## **Lab 3: Exploiting NoSQL injection to extract data**

In this lab, we have to extract the administrator password by exploiting the user lookup functionality.

First, I log in as Wiener, and on Burp’s history tab, I saw `/user/lookup?user=wiener`  GET request

then I checked it by using `'`  and confirmed that the application is not sanitising input.

Then I check if it is accepting Boolean conditions or not. then I enter **`wiener' && '1'=='2**` at user parameter. It returns `Could not find user`

then I change the value to **`wiener' && '1'=='1` this result wiener user details.**

this means we can use Boolean conditions to change the response and retrieve info.

Now, I have tried to find the length of the Administrator’s password by using the following payload:

**`administrator' && this.password.length < 10 || 'a'=='b`**

by try and error, I got the password length which is 8 character.

Now, it’s time to find out the actual password to find that I use **`administrator' && this.password[§0§]=='§a§`** 

and I use Burp Intruder with **Cluster bomb attack type. (Because we have 2 payload positions.)**

After the intruder stops, I note down the administrator's password.

Finally, I successfully logged in as an Administrator.

---

## **Lab 4: Exploiting NoSQL operator injection to extract unknown fields**

In this lab, we have to log in as a Carlos user. For that, we have to first find out the field names and after that the token.

First, when I use the username Carlos and the password **`abc`, it** gives the result `Invalid username or password`

also, I clicked on forgot password functionality it generated the token.

 I checked whether  the password parameter is vulnerable to NoSQL or not by using **`{"$ne":"invalid"}`**

and it gives a different result: **`Account locked.`** This means it is vulnerable.

After that, I add an additional parameter **`$where`** to the JSON data as follows:

**`{"username":"carlos","password":{"$ne":"abc"}, "$where": "0"}`**

to check if the application is vulnerable to JavaScript injection.

After confirming the application is vulnerable, we can use JavaScript in the $where parameter

I use the following payload to find out the fields:

**`"$where":"Object.keys(this)[1].match('^.{§0§}§a§.*')"`  (Use Burp Intruder with Cluster bomb attack** **)**

I got the following fields:

**`id, username, password, email, resetToken`**

I used the following payload to find out the resetToken value:

**`"$where":"this.resetToken.match('^.{§0§}§a§.*')"`**

Finally, I got the token. I used it to request: **`GET /forgot-password?resetToken=13c059be7ed9d03b`**

A page opens with a new password and a confirmed password field. I set a new password for Carlos abc123 and logged in as a Carlos user.
