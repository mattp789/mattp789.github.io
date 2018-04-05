## Pentest Basics

This is my first box that I have gotten root on with no walkthroughs, so I am pretty excited!

I am going to walk you through my process from discovering the system "vtsec" to getting root, and I promise I'll cut out all the boring and frustrating parts!

### Discovery

After setting up the Machine in virtual box, we need to find out what IP the box has.
using netdiscover -r {ip range} gives us a list of hosts on our network.
  
![Image](https://mattp789.github.io/images/discovery.png)

### Enumeration
Now it's time to Enmuerate some services and possible vulnerabilities. First lets run our nmap scan.

I had originally run a more verbose nmap scan, but for the sake of time (and this write-up) I will run a typical -A scan since I know there are no high ports open on this box.

![Image](https://mattp789.github.io/images/nmap.png)

So, we see ssh, ftp, and apache open. I found that using "anonymous" to open an FTP session worked but I got "331 Anonymous login ok, send your complete email address as your password." I could not find a way to use a password as a logon, so I decided to move onto attacking port 80.

LET'S BROWSE TO THE SITE!

![Image](https://mattp789.github.io/images/webpage.png)

What do you mean there's nothing here? I don't believe that for a second. Let's hit the page with "Nikto" and "Dirb" and see what we get.

![Image](https://mattp789.github.io/images/Screenshot%20from%202018-04-02%2017-54-54.png)
![Image](https://mattp789.github.io/images/Screenshot%20from%202018-04-02%2017-56-20.png)

Now we are getting somewhere! We see there is a /secret directory, and on top of that we see /wp-admin! Let's browse to that. Since we are still enumerating things, let's probe around in the secret directory. to our luck we find a post by the user "admin".

![Image](https://mattp789.github.io/images/user.png)

I'm no expert, but I think it is safe to say that this is our admin user. So we can take that over to the wp-admin page

![Image](https://mattp789.github.io/images/admin.png)

I was going to do some brute forcing with hydra or wpscan, but after getting frustrated with hydra, I decided to try and guess the password....turns out it was admin. No one said you had to be a security expert to root this box.

Now that we have access to the administrative page of the wordpress site, we can probably guess that it is somehow vulnerable. My first guess was a reverse shell injection and it turns out I was right!

Thanks to top-hat-sec.com, I was able to spawn a reverse shell using the tutorial at http://forum.top-hat-sec.com/index.php?topic=5758.0

With my new found admin credentials, I was able to upload the fake theme using the provided .zip file from the tutorial. 

![Image](https://mattp789.github.io/images/fake_theme.png)

Edit the Theme Header to send the shell to your IP over whatever port you choose.

![Images](https://mattp789.github.io/images/set_theme.png)

Now, let's open a netcat listener so we can catch the shell the fake theme is going to send us. Are you starting to get tingly yet?

![Images](https://mattp789.github.io/images/netcat_listener.png)

Go back to your WordPress themes and click Live Preview on the Fake theme, and you SHOULD catch a reverse shell.

![Images](https://mattp789.github.io/images/rev_shell.png)

We got shell!!! This is great, but we can't really do much because without a tty shell we can't do a lot of tasks we'd like to do. According to the same tutorial from before we can run the following commands to upgrade our shell:

python -c 'import pty; pty.spawn("/bin/bash")'

export TERM=linux

![Images](https://mattp789.github.io/images/upgrade_shell.png)


After poking and prodding around for far too long, I nearly screamed out loud "LOW HANGING FRUIT!" Let's see if we have access to the passwd and shadow files. WE DO!

$cat /etc/passwd

![Images](https://mattp789.github.io/images/passwd.png)

$cat /etc/shadow

![Images](https://mattp789.github.io/images/shadow.png)

We can see we have a user marlinspike. I copied the output of the passwd and the shadow to a file and ran unshadow to get it ready for John the Ripper.

Unshadow takes the input of your passwd and shadow files and outputs it to a file ready for John the ripper to crack.

![Images](https://mattp789.github.io/images/unshadow.png)

Now that we have the unshadowed file, we can run John on it and hopefully get our password for marlinspike.

![Images](https://mattp789.github.io/images/john.png)


Of course the password is marlinspike...makes all that work seem a bit trival, but oh well this was great practice!! Let's open an SSH session as marlinspike to see what we can do.

![Images](https://mattp789.github.io/images/login.png)

Now it's time to look for some privelege escalation vulnerabilities in this Linux distribution....but first, we may be able to save ourselves some effort by seeing what sudo priveleges Mr. Marlinspike has.

![Images](https://mattp789.github.io/images/sudo.png)


ALL:ALL that means we should be able to su to root

![Images](https://mattp789.github.io/images/root.png)

And consider it PWNED!!

Thanks for reading. I had a blast testing this box. My goal here was to not use metasploit in any way and not look at anyone else's walkthroughs for this box. 

Hope you enjoy. Good luck on your next Hack.


