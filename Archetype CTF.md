# Archetype Box
This CTF was marked as easy. It was a Windows machine running MS SQL
It wasn't as easy as I had expected, as the AV blocked some of my attacks.
However the passwords were available on the machine in clear text, meaning once discovered the machine was compromised. 

I had to install the following tool set on my Kali box to interact with MS SQL
https://github.com/SecureAuthCorp/impacket

Other than the above tools, I used Kali linux's built-in toolset.

## Lets Begin
The target responds to ping

![41ef62304b32471daf1701e9174a4d3c](https://user-images.githubusercontent.com/60744763/130185191-f30edeab-d229-4551-8523-9c307202f083.png)

Now to nmap...

![e4b8dad2ec764d0d8b6f0cb7495f78cd](https://user-images.githubusercontent.com/60744763/130185241-1bd126d5-60f9-46e6-9d40-b50d7cea54e8.png)

Got some interesting things we can look at.

![22553d883ed748d28a890c426477dc2b](https://user-images.githubusercontent.com/60744763/130185275-9612ff39-a85a-47ed-8017-8d3d1756d560.png)

The nmap vuln script found a possible vulnerability with MSSQL

![a19a22ffa48e4b81920328160a598363](https://user-images.githubusercontent.com/60744763/130185314-ac93f941-6140-421a-92d0-17bb188df2f4.png)

First I'll check out if we can access any shares.

![e69f5f407dcc4343936fac1b2514b643](https://user-images.githubusercontent.com/60744763/130185345-2f9d0106-ffe8-4b4f-9456-2ffb1965ac36.png)

backups looks interesting

I can mount it which is a good start.

![e8a8879578b845b1bf04d27d420341dd](https://user-images.githubusercontent.com/60744763/130185382-ecb76cd9-3d09-4957-9a01-075ea772f790.png)

I'll grab the dtsConfig file and see what's in it.

Looks like we have SQL credentials here: sql_svc M3g4c0rp123

![f7a00c8f5fcd4292b63a9f149a4b2245](https://user-images.githubusercontent.com/60744763/130185412-bd02bca1-3bca-4b39-982a-4f0d1a0efb46.png)

I'll also check metasploit for that vulnerability we got from nmap.

![b7ccc35136994f97a6aaceabd68f1752](https://user-images.githubusercontent.com/60744763/130185433-fbb38d6e-03a2-4b40-bf34-1e785631d0f9.png)

We have something here to try... That exploited didn't work. Seems too old.

So I'll try to login use the MSSQL Payload in metasploit with the credentials we found.

I've made the following option settings and the domain set as ARCHETYPE as per the DTSconfig file.

![b9720d88d91e423a869528992d040797](https://user-images.githubusercontent.com/60744763/130185450-a9e77211-a690-404f-bc84-0ac136bcfcd7.png)

It took a few minutes, but unfortunately it failed.

After some research I found this tool set which includes an mssql client. I'll install that and see how I go.

https://github.com/SecureAuthCorp/impacket

So that worked out better, now I have a sql shell

![f428796ffd0144b99ef0057980852e96](https://user-images.githubusercontent.com/60744763/130185478-16d749de-3ad8-42d3-92f5-0b7e33cb14e7.png)

I'll research how to get a cmd prompt from here.

So using the xp_cmdshell option in this tool I've been able to start navigating the system, so will hunt for the first flag.

![fb740e9198fb47abacfa6cf85977c682](https://user-images.githubusercontent.com/60744763/130185510-8ab0eff5-780a-4a34-8d19-b7f777b9fef7.png)

Found it!

![bed743289e5e4e36b19ab03a424eb074](https://user-images.githubusercontent.com/60744763/130185539-64ddc76d-eb04-40ca-89e2-f816f48ff95a.png)

Now to find a way for privilege escalate. I'll try to get a reverse shell first.

I'll make a shell exe with msfvenom

![b0f30d7ff4e547039d3dfd1f32c4920d](https://user-images.githubusercontent.com/60744763/130185562-9a4822dc-33fe-41b7-b393-9f89e5f11f4c.png)

Now download it to the host using a python http server and powershell wget.

I had to do a bit of research to get the 'wget' equivalent working in powershell but found this which it seemed to work.

![3ae0d53584274c9b914f578967507aa1](https://user-images.githubusercontent.com/60744763/130185584-aa672355-f01b-47ad-bf42-6a4793a03a7d.png)

My python http server got a hit

![7ea9fb493ecf4775b77c8358102c013a](https://user-images.githubusercontent.com/60744763/130185600-c7511d0a-c7aa-4bfc-bd0d-cb10f29df9b1.png)

That didn't download, I forgot to specify the output file location.

![389b3f040def44969cc07284020cebb5](https://user-images.githubusercontent.com/60744763/130185616-843ca396-f989-4e94-bf24-793f83bd8b27.png)

Well... that didn't work either. The file is getting deleted as soon as i execute it. Must be windows defender I guess.

I tried a one liner powershell script I found online (link below), it's still getting blocked.
https://hackersinterview.com/oscp/reverse-shell-one-liners-oscp-cheatsheet/

![e292a461f805456baa1a12407c9b2e36](https://user-images.githubusercontent.com/60744763/130185642-22501962-9687-4b21-a0ea-faa112aa0f5b.png)

At least this time it confirmed my suspicion, the AV. I'll have to keep researching for more methods.

I ended up getting a reverse shell by uploading netcat and running the following:

![0ebdefe8b3cc4a2888f60ad2dcfa811c](https://user-images.githubusercontent.com/60744763/130185660-57a84cfd-bc52-4e64-a17c-99b5b1a0775d.png)

Here is my reverse shell from my netcat listener on kali.

![057e45824fa84fa299f3ef59e9a80dcf](https://user-images.githubusercontent.com/60744763/130185687-b82b15f9-f62c-44b1-ac25-69e13880e89c.png)

After some hunting I found the admin's password in the sql_svc user's powershell command history. In hindsight, I wouldn't have wasted time getting the reverse shell working and just hunted around using the SQL shell... Note to self, check user command history first (not last!) ;-)

![3cf147a818ea4c268154f37fdfc73b6a](https://user-images.githubusercontent.com/60744763/130185716-84be74a0-d7dd-4d8f-91c8-32064ea86a0a.png)

administrator MEGACORP_4dm1n!!

I'll try and run a command prompt as admin now.

Because it's a simple shell I cannot get a password prompt with runas

![3a702c69fe0b42b3913b29602f005a72](https://user-images.githubusercontent.com/60744763/130185729-38d6862b-f448-4d4f-b725-2d4f17f46684.png)

After this I tried using winexe in Kali to remotely execute a command as administrator.

![ab71111778654fc6a14ece008f6ec31e](https://user-images.githubusercontent.com/60744763/130185738-46308712-291c-4d28-8bf2-aa6a66b57b96.png)

It responds a bit funny (repeating all my keyboard strokes) but works for this purpose!

And here is the flag!

![ad44ed39d71f43c0aca6c50d622c9c90](https://user-images.githubusercontent.com/60744763/130185759-6c57a3e2-49e2-4ed3-a637-dc2faf75f44e.png)
