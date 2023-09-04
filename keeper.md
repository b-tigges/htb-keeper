# keeper.htb 🐱‍👤
## ! need to add screenshots !

to begin, i started by running a service scan with nmap

Looks like only 2 ports are open.
> 80 : http
> 22 : ssh


OpenSSH is being used, they’re running 8.9 which is a bit outdated, maybe there’s a vuln in there… but let's look the nginx first.

NOTE: Add the hostname from the nmap scan (keeper.htb) to /etc/hosts

The site resolves to this. Lets add tickets.keeper.htb to our hosts as well and see where it takes us…


Tickets.keeper.htb resolves to a login portal. Looks like the site is using Best Practical’s Request Tracker 4.4.4

Perhaps we can use the default RT 4.4.4 creds to sign in?

Digging through RT’s docs I found some default creds. 

root:password



Nice.

Lets see if these creds also work with SSH perhaps?

No luck…







Poking around a bit, I found some other users in Admin -> Users -> Select


Looking at the user ‘lnorgaard’ there’s some hardcoded creds…

lnorgaard:Welcome2023!

Lets try to ssh with these credentials?

Nice, successful connection and we have a user flag.







Now, to get root… Let's see what ‘lnorgaard’ can use as su, if anything.
Try: sudo -l

No luck.

Soooooo let's see what’s in the zip instead.
Try: less <file>


Looks juicy. Lets see if we can extract it to our host.
Try cURL, WGET, SCP…etc

I used scp in this case.
(scp <user>@<ip>:<file> ./<out-file>)

















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