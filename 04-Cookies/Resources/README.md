# COOKIES

## What is a cookie?

It's a simple text data we use to manage session in a website.
When you navigate on a website and click to charge another page on the website, it is not able to know if you are already connected and who you are. The cookie is here for that, to help the website to know who is here. He will send a request to the server to check if the cookie value is assigned to an user or no to be able to display personnalize infos.

The HTTP protocol is 'stateless', every request is independent, the server doesn't remember between two pages. Without cookie the server will forget you just connect when you go to another page.

For example with a mail box, if there is no cookie and you try to access to your mail box he will probably ask you to connect.
If there is a cookie, he will display personal informations from the account associated.

Typical cycle: \
.connexion (username + password) \
.server check → correct ✔️ \
.server answer → "Set-Cookie: session_id=abc123" \
.browser keep it \
.now on every page the browser send automatically: "Cookie: session_id=abc123" \
.the server read this cookie to know who you are


## How to find the vulnerability?

Open your console with 'Inspect', go to 'Application', check in the cookie board if there is one. \
Here you can find one named 'I_am_admin' with a matched value.

Name: \
I_am_admin

Value: \
b326b5062b2f0e69046810717534cb09

The value is a hashed MD5 word. \
*More infos about MD5 hash in the general README*

When you pass this value in a MD5 hash lookup tool he will say 'false'. \
It means the value say 'no i'm not the admin'!

What will happen if you try to pass 'true'? \
So now you do it the opposite and hash the word 'true' and change the cookie value with it and actualize the page and that's it!

The server do probably : \
if MD5(cookie) is equal to the hash of "false" → classic user \
if MD5(cookie) is equal to the hash of "true" → admin

The flag appear because you literally send to the server 'yes i'm the admin' with your console!

This type of attack is called *privilege escalation*: gaining higher access rights than originally granted, by manipulating data the client shouldn't be trusted to control.


## Suggestions to protect cookie

### Why it was a vulnerability?
The cookie directly contained the privilege information `I_am_admin=false`. \
The browser has complete control over the content of a cookie. \
The server trusted this value without verifying it against a reliable source. \
Changing the cookie was equal to changing your own privileges! \
MD5 hashing is not designed to protect anything public, because anyone can calculate it now.

### How to do protect better?
The privilege information itself must not be store in the cookie. \
The cookie should only contain a randomly generated and unpredictable session ID, impossible to guess or recalculate.

The real information must be always store on the server side. The server keeps, in its own database, the association between the session ID and the corresponding user. The client must not be able to see or control this association.

### Conclusion
The server delegated the decision regarding its own privilege level to the client (using a cookie), whereas this decision should always be verified and stored on the server side. A simple MD5 hash, which is now calculable by anyone, does not constitute protection for sensitive data. \
This echoes the OWASP category A01:2021 – Broken Access Control, which deals with the fact that access controls are not being properly enforced on the server side.
