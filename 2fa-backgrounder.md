# What is Two Factor Authentication, and Why am I Ranting About it?
### About 2FA
Two-factor authentication (2FA) is a thing that's [all](duosecurity.com) [over](authy.com) [the](https://www.google.com/landing/2step/) [place](https://support.apple.com/en-us/HT204152). Companies like Apple and Google want to make sure you are who you say you are, so they like 2FA.

So, why is 2FA better than just a plain ol' password? Well, it's in the name. Instead of only using one thing to verify your identity (your password), 2FA processes add a *second factor* -- often augmenting "something you 'know'" (your password) with "something you 'have.'"

### Types of 2FA

There are a bunch of different ways to get that second factor. The most common one is a cell phone (heck, that's what 2FA provider [Authy](authy.com) is perhaps best known for! Especially now that they're part of the [Twilio](twilio.com) family). Even if you're not familiar with Authy, you might have received one of those numeric codes that gets sent to your phone every once in awhile to make sure you are still the one accessing your Gmail account. Another way to do 2FA uses [physical objects](inception totem!) to sign you in.

The trick is that these hardware tokens are rather expensive and unfun to use. Much like the text message option, you'd push a button on the token, and enter in a long number, and then it would let you in. Rather comically, it was easy to misread numbers on some of these tokens, making for a really crappy experience. :( 

Both Duo and Authy have ways of replicating a hardware token's benefits in a smartphone app: namely, you can approve or reject a login request at the push of a button. Pretty cool, eh? Yubico, though, attacks this from a different perspective. 

Yubico's tokens (often called YubiKeys) plug into your computer's USB port, and generate a code that can be used to identify you to the application (just like Duo and Authy's pushed authentication requests).

### Why 2FA?

Because of the huge security benefits that 2FA provides, all the big names are using it. But a bunch of smaller companies, and even individuals (victims of in-person and online harrassment, people whose identities were stolen, or simply security-conscious people) use 2FA to get that extra assurance that they will be the only ones accessing their accounts and resources.

It would make sense that two companies -- GitHub and Yubico --  deep into the world of enterprise software (read: huge-organization-with-lots-of-people-and-lots-of-politics) would be making this easier for the average person to consume. That's where the GitHub YubiKey comes in.

![image of YubiKey]()

I picked one up ASAP, and it arrived late last week.

### What can you do with a silly lil USB key?

There are a ton of things you can do with it, but the simplest one is to secure your GitHub account with a super-easy second factor -- a USB key that fits on your keychain.

I also set up one of my computers to use the YubiKey to log me in. [Check out this post for more info.](link-to-below)

--------------
