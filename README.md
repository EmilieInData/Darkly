# Darkly
Introduction to cyber security, general vulnerabilities we can encounter on the World Wide Web.


You will find below some general informations to help you to understand all project details.

## What is OWASP — Open Worldwide Application Security Project?
//to fill


## What is a MD5 hash?
This is the result of a hashing algorithm called MD5 (Message Digest Algorithm 5). \
It takes any data (text, file, image..) and transforms it into a fixed-length digital fingerprint.

<pre>
forty-two → a16ca146e5e432fa7e30319121ea7473
</pre>

→ Characteristics: \
.Always 32 hexadecimal characters and only uses '0123456789abcdef'. \
.Each input always has the same hash.

→ How to distinguish it from other hashes? \
<pre>
Algorithm      Length
MD5            32 characters
SHA-1          40 characters
SHA-256        64 characters
SHA-512        128 characters
</pre>

It was theoretically irreversible, the original data could not be directly recovered from the hash. \
But today it is considered cryptographically broken and is no longer recommended for security because many databases of known hashes exist and it has become very easy to decrypt MD5 hashes.

https://md5decrypt.net/
