# Oopsie Box

This challenged didn't have much information other than the IP address.
As it turns out it was related to the previous challenge Archetype. In that it used the same admin password. 

For this challenge I used Kali linux tools, and OWASP ZAP quite a bit.

## Recon

I'll start off with nmap

![ed5266c6ae315c82ba7399eb87157850.png](images/dc097765129245e39494d93dfa8dc270.png)

We have ssh open and http

Brings us to a nice web site

![7a1061154de38280c9da6c9127abeb3c.png](images/892fc5f7a0d84c99bc6826d6314ebba2.png)

In the page source we can see some other folders containing scripts and a potential login portal.

![ad30a17813e62a09b03cb5b5344df8ad.png](images/d2dfa9a7d1e247e8a6712ee9df35e7ca.png)

Found a login page by using the above URI

![1e68f6183ab81b3cbbca991a0df66659.png](images/6943ae6f70e046d187bd5faaefc794d7.png)

## Compromising the server

The next part was tricky, I tried some injection commands and brute force attacks, but wasn't getting anywhere.

After reading on the forum, it turns out the password was the same from the previous CTF. To be honest I thought the challenges were not linked, but in this case they are...

After using that password I got from the ARCHETYPE CTF with the username admin I was able to get access to the site. 

First thing I noticed was an uploads section which should be a good place to upload some code, like a reverse shell. But it requires 'super admin' rights to the site.

![1fc5cbad859d02994e2da69185b24b85.png](images/933f15c3295545d5957cdb1199da7673.png)


## Payload delivery 
## Exploiting IDOR, an Upload portal and cookie IDs.

Under the accounts section we can see an IDOR referencing the user ID, maybe I can find the super admin's ID.

![657a801cd0cc650ff13f4e37241f4ee1.png](images/3c8fc02b388546bfa2543c07e79cf56e.png)

Also in a request after I logged in, I can see the cookie matches the 'Access ID' in the Admin's account. I'd suggest if we can find the super admin's ID, we could use this as the cookie information.

![fe750dcf640a8cfb053b89279be71042.png](images/efdcf7eacfaf4646bc8835b5d6ee860c.png)

Trying a few ID numbers led to no info, but I will try and brute force these numbers with the ZAP fuzzer attack.

![59b6d9c18a720bc0a9eeeeaf1baf9490.png](images/fe43cc815815497fad6c30b155a1f3dd.png)

I'll try ids from 1 to 100 using the numberzz feature in the fuzzer. 

![a411504b4c79b332f706a88b9d344565.png](images/5d5da03b22294cef9361c55dd5624990.png)

After a few seconds I had a hit, ID 30 had a similar body response size as ID 1. 

![7d16612a9ccc47c4a663535dc35e0195.png](images/65cb044acadc4f9491cb9c27d9ca5dde.png)

Now I have the super admin's cookie id, their Access ID.

![2bddbd0a719ee462e8f652ba5f8af825.png](images/8fd134815bd142c8b52c40639594fc48.png)

I turned on proxy intercept (or break mode) and changed the cookie ID to try and access the upload section.

![10b163d95f41f7f721a176656275f02b.png](images/53ff843bc9464b0599ada5e2038d6f9c.png)

Now I have access to the Upload portal. Next I'll upload a simple PHP reverse shell and create a netcat listener on my attack box.

Upload successful, I had to intercept the request and update the cookie as per the last step too, otherwise I was getting access denied. I also had dirbuster (DIRB) running in another shell when I was trying to brute force the admin login page. DIRB found the uploads/ URI, but I accidentally closed it before taking a screen grab.

Now to access the uploaded file.

![b2bc8d6d11298416ae0fdbc7637eb4ff.png](images/cb03da13cd824cbb9258d85406a5e845.png)

And I have a shell!

## Flag 1

A quick search around the home folders and I found the first flag.

![9c5764b45f45d0c7869539770c2086b2.png](images/43b8523d744c4b0ba42bbc99e205b2df.png)

## Privilege Escalation

I did a search for SUID files with find / -perm -4000 2>/dev/null and found one called bugtracker, but I don't have execute rights as my current user.

![96d3f77d9eccde28cc616cac4b418e00.png](images/2c30a5bfa49f42419c10efdf46069a91.png)

I'll hunt around the www folder, since the site probably uses a DB, there might be something there.

We are in luck the db.php file contains some user credentials, the user who we found the first flag in.

![d1fcda2557d66427155a882375736206.png](images/148cc9ace71f457b8766da0e16483624.png)

I'll try this over ssh.

![4883769d5553759baae43f1fe3a237a6.png](images/54dda1d065354985aa1cb63600226413.png)

I'm in, and it looks like i can run the bugtracker application as i'm in the appropriate group. I noted that I cannot run sudo as I'm not in the sudo group.

So I'll check out bugtracker to see if we can exploit it. 

A nice little shell app, looks like it retrieves some data from somewhere.

![54e6f1c2217b754f8741d0e0ecb951d6.png](images/e79558a1cede45378091c5c2f1828b65.png)

I'll see what the strings command yields from this executable.

![fd8bc46bf387c3c5283ef1370ec3f851.png](images/50323afe80f54f47af365f85674e00dc.png)

The program makes a call to 'cat' using a relative path, so I can exploit this.

I'll create a file called cat that launches a bash session and adjust my path.

Here is the file, permissions updated to executable and path adjusted.

![526a94e2257c2643994d627499a47a05.png](images/5c09336a9ad14d9783f3ab30130ab481.png)


## Flag 2

And here we have it! I am root, below is the flag

![cf51c7fb86063f867562d62c05c859f7.png](images/b278382ac2f14e98a0da721c67958fd6.png)
