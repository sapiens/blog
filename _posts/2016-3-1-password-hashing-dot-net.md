---
layout: post
title: C# (.Net) Password Hashing The Easy Way 
category: .net
---

We know we shouldn't store passwords at all, not even encrypted nevermind in plain text. And by encrypted I mean something like AES, Base64 encoding is _not_ encryption. In a nutshell, we need to store only salted hashes so that a password can't be reversed and an attacker shouldn't find a common pattern that will lead to breaking the password. As any developer knows, we want a hash which is slow to generate, so fast algorithms like MD5, Murmur or the SHA family shouldn't be used. For best results something like bcrypt or Pbkdf2 should be used with a high number of iterations. 

In order to make things easy, I've come up with a value object that does the 'hard' work and it's versatile enough to allow you to customize things. It's a class called `PasswordHash` available as part of my [CavemanTools](https://www.nuget.org/packages/CavemanTools/4.0.0) library or you can copy/paste the code directly from [here](https://github.com/sapiens/cavemantools/blob/master/src/CavemanTools/PasswordHash.cs) .

First thing is to decide on the hash length and I suggest to set a value > 32. This value should vary from one app to another and should be secret. I also recommend to set a (secret)salt size and a (secret) iteration count.

```csharp
PasswordHash.KeySize = 40; //default is 32
PasswordHash.DefaultSaltSize = 32; //default is 32 
PasswordHash.DefaultIterations = 70000; //default is 64000
```

Be aware that the number of iterations affects the hash generation time. The default is a good value (it takes around 200ms on my PC) but you might want to increase that for sensitive apps. 

Let's use it

```csharp
var pwd = new PasswordHash("some pwd");

//store as bytes
store(pwd.Hash);

//store as string
store(pwd.ToString());

//restore from string
var pwd=PasswordHash.FromHash(hash,PasswordHash.DefaultSaltSize,PasswordHash.DefaultIterations);

//restore from bytes
 var pwd = new PasswordHash(hash, PasswordHash.DefaultSaltSize, PasswordHash.DefaultIterations);
 
 //check pasword
 if (pwd.IsValidPassword("other pwd")) { }
```

### Customizing how the final hash is generated

Regardless of how you keep it in the database, it's important to be aware that the final hash is a combination of salt and the password hash. By default, salt is stored first then the password hash. And that's why we want to change the passwords hash length or/and the salt size, so that an attacker would have a hard time guessing when the salt ends and when the hash begins. If an attacker knows either, they already have 2 pieses of the puzzle. 

Another way is to change how the salt and hash are stored. 

```csharp
PasswordHash.PackBytes = (data) => /* return a byte[] containing the salt/hash stored in a creative manner*/;
PasswordHash.UnpackBytes = (packed,saltSize) =>  /* 'recover' the salt and password hash */;
```    

Basically, we have these options to ensure an attacker won't have an easy job:
* Variable/Secret key (password hash) length
* _and/or_ Variable/Secret iterations count
* _and/or_ Variable/Secret salt size
* _and/or_ Your own 'pack/unpack' algorithm

This way, even if an attacker knows you're using `PasswordHash` they still have to guess the above.

