---
title: secure passwords for the masses
tags:
    - security
    - dev
    - crazy
categories:
    - dev
main_img:
    path: better_passwds.jpg
    copyright:
        name: Dev.Arka
        url: https://www.flickr.com/photos/totallygenius/
---

##whats the problem?
there are tons of services we use. every one of them needs a login and password. we either have to #1 use one supersecure password for all of them or #2 we have to create and remember tons of good passwords. #1 is ok if we can be sure that none of the services we use will ever be compromised. and as we do not live in a world where this will ever be reality we have to go with option #2. but what do we do if we do not even remember if we had to buy milk and/or fish? and how do we create strong passwords?

##what can we do about it?
i can think of some solutions for that:

###using a sheet of paper inside your wallet
schneier once stated that [one should write down the passwords](http://www.schneier.com/blog/archives/2005/06/write_down_your.html) so we can choose non-dictionary passwords that are hard to remember. and because we are used to handle important things inside our wallet this should be a good place to keep this paper. check out the comments below the linked post for some ideas of password obfuscation when writing them down. i think its not a total bad idea but i'm to lazy to write all this down. especially if i'm not sure if i'm going to use the service i just registered to in the future. and i have to figure out a way to create passwords, too. just. too. lazy.
###using an extra service/software
there are some apps and services out there which help us store multiple passwords accessible via one masterpassword. i've always been a little sceptic about software handling my passwords. what if theres a backdoor or just a major bug in the latest update? should never happen, but what if? but stored passwords (e.g. in the browser) are best for the laziest ones. on the long run i started using them simply because its so damn easy. if we want this comfort on other systems then our own we have to switch to something like password managers on usb-sticks. i never checked those out because of laziness and i never wanted to buy extra hardware. what if i loose it? does it run on all machines/systems..? 
###using algorithms
we can try to create algorithms that use things we know (lyrics, citations, ...), change some chars to 1337-speak, combine them with parts of the service (domain-)name (this way we get a unique password for every service) and hope that we'll [remember this monster](http://xkcd.com/936/). 

###using hash-functions
another way of finding a unique password for every service that we can easy remember is done by using hash functions. there a plenty of them out there for different usecases. [those used in a cryptographic context](http://en.wikipedia.org/wiki/Cryptographic_hash_function) must withstand some extra requirements. some of them have proven strong others don't. hash functions are often used to get sort of a small fingerprint (size depends on the used hash function from) of large data (like binary files). one of those special cryptographic requirements stands for the fact that one should be able to "unhash" something. so nobody should be able to recrate the initial data using the hash. another interesting thing about hash functions is the fact, that the fingerprint-strings (the hash) length is always the same. if we for example use sha512 to create such a fingerprint from a very small amount of data, say the string "password", we get the hash "b109f3bbbc244eb82441917ed06d618b9008dd09b3befd1b5e07394c706a8bb980b1d7785e5976ec049b46df5f1326af5a2ea6d103fd07c95385ffab0cacbc86". let's use it as our new password! what we just did was pushing the [entropy](http://en.wikipedia.org/wiki/Password_strength#Entropy_as_a_measure_of_password_strength) from 28.7 bits to 508.3 bits. now we add some additional info to the masterpassword-string that we hash to make it unique per service like "password@mail.google.com". the new hash is "e87aa5d1bb0d29491fd7f2161f9c3f1d1cfa6bc84bc76c8dec705cb5e200fa3906ba576e8988f0441b757c1ecd8b39c2357f8b1fd3f28fde998d88bb9d1e8aa6" and totally different from the first. this way we can create (and recreate) really strong passwords for every damn service out there. if the service can't handle passwords of this length we take the first n-chars (with n being the max password length). there is no need to remember those passwords as we simply use the same hash-function again. we only have to remember:
- our masterpassword
- a very simply concatenation algorithm of the servicename and the masterpassword 
- the password length per service (to avoid this we could always simply stick with 16 chars)
- the used hashfunction

as the whole strength of this method lies in the masterpassword we should choose this one password wisely. if done so we can even write the service-strings (the stuff we concat our masterpassword with per service) down (maybe even including the password length if needed). remembering the used hashfunction should be easy if we always use the same.

the fun thing about this is, that those hashfunctions are so damn popular that we can find them simply everywhere. we could use [online-hash-services](http://caligatio.github.com/jsSHA/) (the hash is calculated on the client via javascript, so our masterpassword stays with us). another simple way would be the use of the sha512sum in your shell (if you have one :-P) like this: 

    echo -n 'masterpassword@google.com' | sha512sum

. thats it. if we only need the first 32 chars we simply could do it with 
   
    echo -n 'masterpassword@google.com@32' | sha512sum | cut -c1-32
.

Update:

Check out my [script](/assets/static/hashed_passes.html). It generates passes from strings with the format 'masterpass@service@passlength' on your client. enjoy.
