# keeper.htb 🐱‍👤

to begin, i started by running a service scan with nmap <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/nmap.png "initial nmap scan")<br>
######try: nmap -sV <ip>
looks like only 2 ports are open. <br>
> 80 : http <br>
> 22 : ssh <br>


OpenSSH is being used, v8.9 which is a bit outdated, potentially vulnerable… but let's look at nginx first. <br>
**NOTE:** add the hostname from the nmap scan (keeper.htb) to /etc/hosts <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/website.png "website langing page")<br>
the site resolves to this. lets also add tickets.keeper.htb to our hosts as well and see where it takes us <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/login_portal.png "login portal")<br>
tickets.keeper.htb resolves to a login portal. the site is using Best Practical’s Request Tracker 4.4.4 <br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/wiki_page.png "RT documentation")<br>
perhaps we'll get lucky and we can use some default RT 4.4.4 creds to sign in? <br>
some quick digging through RT’s docs I found some default creds. <br><br>
> root:password <br>
lets try the default creds. <br><br>
[!alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/login_as_root.png "default creds") <br>
nice <br>
<br>
soooo next up lets also check if these creds work with SSH ? <br><br>
[!alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/ssh1.png "ssh fail") <br>
no luck. <br>
<br>
poking around the RT panel a bit, i found some other user account information in Admin -> Users -> Select <br><br>
[! alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/users.png "user accounts") <br>
looking at the user ‘lnorgaard’ there’s some hardcoded creds in his user description…
[!alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/user_comment.png "user description")
> lnorgaard:Welcome2023! <br>
nice, maybe we can use these to login to the site? <br>
but first im interested in trying to ssh with these credentials. <br><br>
[!alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/ssh2.png "ssh success") <br>
nice! successful connection and we have a user flag. <br>
<br>
now that im on the machine lets see what ‘lnorgaard’ can do as su, if anything. <br><br>
######: sudo -l <br>

No luck. <br>
<br>
Soooooo let's see what’s in the zip instead. <br><br>
######: less <file> <br>


Looks juicy. Lets see if we can extract it to our host.
######: cURL, WGET, SCP … etc

I used scp in this case.
######: (scp <user>@<ip>:<file> ./<out-file>)

















Once extracted, we get two files ‘KeePassDumpFull.dmp’ and ‘passcodes.kdbx’
Lets see if we can extract any info from the dump?
I found the following tool on github: https://github.com/vdohney/keepass-password-dumper 
(KeePass-Password-Dumper)
For some reason this shit would not work in my Kali box because of outdated .NET sdk and it was annoying me so I copied the file to my host env, Windows 10.


After running the tool we’re given the following output. This looks like gibberish or a different language?

Goog suggests perhaps there’s a small typo or missing info? Looks like ‘dgrød med fløde’ is supposed to be ‘rødgrød med fløde’ which means Red Berry Pudding (possible passphrase?)







Let’s see if this is the master passphrase for the keepass file. 
I installed the KeePass ‘kpcli tools’ on my kali box to look at the file in my terminal.

Try: kpcli, then open <file> and enter the passphrase when prompted


Nice, looks like it worked lol.

I looked inside passcodes, there’s a bunch of sub-dirs and the only one with info is Network.






Quick Googling of kpcli commands, I found that the ‘show’ command lets you peer into entries.

Looks like there’s potentially some root creds in here?
root:F4><3K0nd!
Lets try to ssh with them now…

Looks like it's not gonna work. However, we do have a PuTTy key here. PuTTy is an ssh client, so let's see if we can make our own key with this info and then try a regular ssh with it?

I made a file called ‘key’ in the .ppk format (private-putty-key) and then from here we need to convert it to a standard ssh usable key format such as .pem, and not the putty-proprietary format for their client.

Using the puttygen utility I created the following pem file.












Now we can try to ssh using our new key.

Looks like it worked! 
We can access the root flag from here. 

Success!
