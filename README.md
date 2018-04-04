## Pentest Basics

This is my first box that I have gotten root on with no walkthroughs, so I am pretty excited!

I am going to walk you through my process from discovering the system "vtsec" to getting root, and I promise I'll cut out all the boring and frustrating parts!

### Discovery

After setting up the Machine in virtual box, we need to find out what IP the box has.
using netdiscover -r {ip range} gives us a list of hosts on our network.
  
![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/discovery.png)

### Enumeration
Now it's time to Enmerate some services and possible vulnerabilities. First lets run our nmap scan.

I had originally run a more verbose nmap scan, but for the sake of time (and this write-up) I will run a typical -A scan since I know there are no high ports open on this box.

![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/nmap.png)

So, we see ssh, ftp, and apache open. I found that using "anonymous" to open an FTP session worked but I got "331 Anonymous login ok, send your complete email address as your password." I could not find a way to use a password as a logon, so I decided to move onto attacking port 80.

LET'S BROWSE TO THE SITE!

![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/webpage.png)

What do you mean there's nothing here? I don't believe that for a second. Let's hit the page with "Nikto" and "Dirb" and see what we get.

![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/Screenshot%20from%202018-04-02%2017-54-54.png)
![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/Screenshot%20from%202018-04-02%2017-56-20.png)

Now we are getting somewhere! We see there is a /secret directory, and on top of that we see /wp-admin! Let's browse to that. Since we are still enumerating things, let's probe around in the secret directory. to our luck we find a post by the user "admin".

![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/user.png)

I am no expert, but I'd be willing to be this is our admin user. so let's take that over to /secret/wp-admin

![Image](https://github.com/mattp789/mattp789.github.io/blob/master/images/admin.png)

Now from here, I was able to login as admin, and I'd love to tell you that is was by bruteforcing, or using any cool tools to get in....but turns out guessing that the password may be admin was correct! No one said this would be too difficult. A lot of my research, time, and frustration went into the next couple of steps.

Now that we are in the wordpress manager as admin, we know that wordpress is probably vulnerable to some sort of reverse shell type of attack. As it turns out it was. 

