# Authentication

## Introduction

Almost all application and systems in today's world relly on authentication, most often made up of a username and a password. Often there is a requirement to have a strong password which meets certain criteria, however just like a chain is as strong as its weakest link, an authentication system only works if it's properly implemented. Although relatively simple, this vulnerability is one of the most critical due to the large threat such a breach could pose. Authentication vulnerabilities occur when a malicious actor is able to bypass security due to weak or broken authentication and get into the system with elevated access. In other words they are able to get into the system or application without authenticating. This allows them to compromise user data, get access tokens and passwords, or worse take control over the whole system. As a result it has been listed as one of the top security vulnerabilities in OWASP Top 10 since 2013. 

A good example for this vulnerability would be a system or application where there are users and admins. Once someone logs into the system as a user a session cookie is generated that is encoded as text, for instance:
```
ewoidXNlcl9uYW1lIjogImpvaG5fZG9lIiwKInJvbGUiOiAidXNlciIKfQ==
```
When this session cookie is decoded the result looks like so:
```
{
"user_name": "john_doe",
"role": "user"
}
```
A malicious actor can then easily change the role to be "admin", encode it and use that as the session cookie for the site. The site will then see them as an admin even though they never actually authenticated through a username and a password. 

In order to prevent this from happening there should be additional authentication checks to block users from accessing the system without supplying the right credentials. Furthermore, details related to the session should not be exposed either in the URL or the cookie in a way that is easily accessible and changeable. Another way this can be prevented is by utilizing multi-factor authentication or allow the sessions IDs to be generated server-side and not dependent on the user details. 

Imagine that there was a site where the preffered language and location had to be set every time that it was accsesed. Not just on the home page, but every link that was clicked on that site would first greet the user with a prompt to select the language and region - this would get quite annoying relatively quickly. This is where cookies come in to play. When a user first opens that site, they select their language and location then the site generates a text file with that preference which is saved on the user's machine. That file is called a Cookie. Now when the user loads a new page on that site, instead of throwing the prompt the site checks the cookie and takes the settings from there. Similarly, cookies can be used when authenticating to a site.


## Examples

An example of how this would look like in code is again through the use of cookies, more especially around how a cookie is created in the backend by a script. In this case there is a function called create_cookie which uses the http.cookies Python library in order to generate cookies. For this scenario the cookie can be considered to be like a dictionary structure-wise and the function builds the keys and values. The main key in that dictionary will be the sessionID which is an argument taken by the function. On its own the create_cookie function is fine, however the issue arises when the sessionID is set as the user_name. In that scenario a malicious actor could guess the sessionID and replace it to impersonate any other user in the system.
As this of course is a simplified example it in no way aims to mimick what a real back-end application would look like.

```
from http import cookies

def create_cookie(sessionID):
    cookie = cookies.SimpleCookie()
    cookie['sessionID'] = sessionID
    cookie['sessionID']['expires'] = 24 * 60 * 60

user_name = "john_doe"
create_cookie(user_name)
```

In order to resolve it the point of failure will need to be fixed, which in this case is the user_name being passed on as the sessionID. Although the vulnerability itself in the code above would be the create_cookie function call, the fix is in the function itself. Ideally, the need for the system to pass the sessionID should be removed. Instead the create_cookies function should generate its own sessionID everytime it is ran, and that sessionID should not be something that is easily guessable. For that a new library will be used called secrets. This library has an option which randomly selects an item from a given array, here the array is called allowed_chars and it contains all alphanumeric characters. Next a random item is picked 32 times and joined together to form a truly secure sessionID.

```
import secrets
from http import cookies

allowed_chars = 'abcdefghijklmnopqrstuvxzy' + '0123456789'

def create_cookie():
    cookie = cookies.SimpleCookie()
    sessionID = ''.join(secrets.choice(allowed_chars) for i in range(32))
    cookie['sessionID'] = sessionID
    cookie['sessionID']['expires'] = 24 * 60 * 60
```


## Practice


# Cryptography

Sending data over the internet or just storing it in general as it was first written in plain text would be an extremly bad idea and practice, since anyone that can access or intercept that data would compromise it. Therefore, encryption is used, which scrambles the data from what it was into something else that cannot be understood unless a key is used to put it back in its original state. A very common encryption method is ROT-13, where each letter in the text is replaced by the letter that comes 13 slots later in the alphabet, for example A becomes N, B becomes O, C becomes P, and so on. This would work for very simple things that won't cause any damage if discovered but as it is quite well known using it on more sensitive data would get that data immeditley compromised. It is here that better encryption algorithms have to be used, such as public-private key based ones or asymetric cryptography.

The insecure cyptography vulnerability occurs when either a weak alogirthm is used to encrypt sensitive data, or the key that was used for encyrption is not stored securiley. 