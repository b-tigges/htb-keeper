# keeper.htb 🐱‍👤

to begin, i started by running a service scan with nmap <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/nmap.png "initial nmap scan")<br>
`nmap -sV <ip>` <br><br>
looks like only 2 ports are open. <br>
> 80 : http <br>
> 22 : ssh <br>


OpenSSH is being used, v8.9 which is a bit outdated, potentially vulnerable… but let's look at nginx first. <br>
**NOTE:** add the hostname from the nmap scan (keeper.htb) to /etc/hosts <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/website.png "website langing page")<br>
the site resolves to this. lets also add tickets.keeper.htb to our hosts as well and see where it takes us <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/login_portal.png "login portal")<br>
tickets.keeper.htb resolves to a login portal. the site is using Best Practical’s Request Tracker 4.4.4 <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/wiki_page.png "RT documentation")<br>
some quick digging through RT’s docs i found some default creds. <br>

> root:password <br>

lets try the default creds. <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/login_as_root.png "default creds") <br>
nice <br>
<br>
soooo next up lets also check if these creds work with SSH ? <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/ssh1.png "ssh fail") <br>
no luck. <br>
<br>
poking around the RT panel a bit, i found some other user account information in Admin -> Users -> Select <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/users.png "user accounts") <br>
looking at the user ‘lnorgaard’ there’s some hardcoded creds in his user description… <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/user_comment.png "user description") <br>
> lnorgaard:Welcome2023! <br>
<br>

nice, maybe we can use these to login to the site? <br>
but first im interested in trying to ssh with these credentials. <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/ssh2.png "ssh success") <br>
nice! successful connection and we have a user flag. <br>
<br>
now that im on the machine lets see what ‘lnorgaard’ can do as su, if anything. <br>
`sudo -l` <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/sudo_check.png "sudo check") <br>
noooooooooo <br>
<br>
so lets see what’s in the zip instead. <br>
`less <file>` <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/files.png "files on box") <br>
looks juicy. lets extract it to our host to further dig into it. <br>
`cURL, WGET, SCP … etc` <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/using_scp.png "using scp") <br>
i used scp <br>
`scp <user>@<ip>:<file> ./<out-file>` <br><br>

once extracted, we get two files ‘KeePassDumpFull.dmp’ and ‘passcodes.kdbx’ <br>
if we're lucky maybe we can extract the master passphrase from the dump to get into the kdbx (KeePass password database) <br><br>
i found the following tool on github: https://github.com/vdohney/keepass-password-dumper (KeePass-Password-Dumper) <br>
#### (for some reason this shit would not work in my Kali box because of outdated .NET sdk and it was annoying me so I copied the file to my host env, Windows 10) <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/keepass_crack1.png "using keepass dumper") <br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/keepass_crack2.png "using keepass dumper") <br>
after running the tool we’re given the following output. looks like gibberish or a different language? <br>
<br>
the Goog suggests there’s a typo in my search? it looks like ‘dgrød med fløde’ is supposed to be ‘rødgrød med fløde’ which translates to Red Berry Pudding (possible passphrase?) <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/google1.png  "google results") <br>
lets see if this is the master passphrase for the keepass file. <br>
i installed the KeePass kpcli tools to easily pick apart the kdbx file in my terminal <br>
`apt-get install kpcli` <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/kpcli.png "kpcli usage") <br>
looks like it worked lol <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/kpcli2.png "kpcli") <br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/kpcli3.png "more kpcli") <br>
inside passcodes, there’s a bunch of sub-dirs and the only one with info is Network. <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/kpcli4.png "more more kpcli") <br>
quick Googling of kpcli commands found that the ‘show’ command lets you look at entries. <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/kpcli5.png "juicy stuff") <br>
looks like there’s potentially some root creds in here? <br>

> root:F4><3K0nd! <br>

lets try to ssh with them now? <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/ssh3.png "ssh fail again") <br>
no luck again. however, we do have a PuTTy key here. PuTTy is an ssh client, maybe we can use this key to ssh ??? <br>
i made a file called ‘key’ in the .ppk format (private-putty-key) and then from here we need to convert it to a standard ssh usable key format such as .pem, and not the putty-proprietary format for their client. <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/key_making.png "epic ssh key") <br>
using the puttygen tool i created the following pem file.<br>
`puttygen <input-key> -O <key-type> -o <out-key-name>` <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/puttygen.png "new pem") <br>
lets try to ssh as root now. <br><br>
![alt text](https://raw.githubusercontent.com/b-tigges/htb/main/screenies/ssh5.png "root") <br>
looks like it worked. <br>
success !
