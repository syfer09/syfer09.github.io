---
layout: default
title: Intro Track
date: 2025-06-03 14:44:00 +0000
categories: jekyll update
permalink: /intro
---

# Intro

The Hack the Box website has a couple of starting tracks with several easy boxes being grouped together. These come with guiding questions, and as this is my first attempts at writing more formal reports I've decided to do a writeup on them even though they are quite simple, each dealing with a single introductory concept. My previous and first post was done on the first box in the first track. This is the rest of the boxes in that first track.

## Fawn

This box deals with the File Trasnfer protocol or FTP for short. After establishing a connection to the target and running an nmap scan we see the following:

![NMAP Scan](/assets/img/fawn/fawn_scan.png)

Port 21 is open and running a service called vsftpd with version 3.0.3. Since 21 is the most common port for FTP we already suspect that, but a quick Google search verifies that VSFTPD stands for "Very Secure FTP Daemon" and is the default FTP server on the most common Linux distributions.

The first step to try here is to make a connection to the FTP Server and see what happens. This is done through `ftp $ip`. This ends up being successful and prompts for a username and password. After trying some default credentials like `admin`, `anonymous` in both username and password established a login. After that, an `ls` shows the flag in the root directory.

![FTP flag](/assets/img/fawn/fawn_ftplogin.png)

## Dancing

The next box is a quick introduction to the Server Message Block protocol or SMB for short. Establishing a connection and scanning the target we see a lot of services:

![NMAP Scan](/assets/img/dancing/dancing_scan.png)

The three open ports are 135, 139 and 445. I know that the msrpc service is Microsoft's implementation of the remote procedure call, and I know that both ports 135 and 139 are connected somehow despite being different services. 

As I mentioned previously, these boxes have guided questions and since the question was asking about SMB, I went at exploiting port 445 rather than looking more in depth at the two other services. Another reason why I didn't give them a closer look was due to seeing them so frequently in scans across a variety of boxes. As something that is present as default it often ends up being the last to be looked at.

Regardless, port 445 has a "microsoft ds?" service running on it which I know from previous experience is an SMB database. Therefore I can skip the verification and go straight into using the smbclient utility to interact with the server. The using the flags `-N` and `-L` we can list the avaliable shares:

![SMBclient List](/assets/img/dancing/dancing_smb_enum.png)

The first three shares are ones that I've seen on every SMB server, and the only non default one appearing to be the `Workshares`. Using the same `smbclient` utility we are able to access the share, which luckily lets us in with a blank password. There are then two directories, Amy and James that I can look around in. After some searching, the flag is in James's directory and `get` command allows me to download it.

![In SMB Server](/assets/img/dancing/dancing_smb_flag.png)

## Redeemer

This is another simple box dealing with an in-memory database system called Redis. This being part of an intro track makes me think it is a common utility, yet this was my first time encountering this particular database.

The scan showed only one open port:

![NMAP Scan](/assets/img/redeemer/scan.png)

I learned that similar to the previous SMB database there is a CLI which allows us to interact with a Redis server called `redis-cli`. Luckily it was already installed on my Kali setup. Using this tool we can request some info to further enumerate the database:

![Info Log](/assets/img/redeemer/redis_connect.png)

There is a lot of useful info here like the version number and the configuration file location, however the most important turned out to be at the very bottom.

![Keyspace](/assets/img/redeemer/keyspace.png)

Unfortunately I would not have realized this was important without the guiding question attached to it. For some reason I did not associate keyspace with being an individual database, despite knowing that Redis followed a key-value structure. Nevertheless, this last line of the log file shows that there is one database with 4 key-value pairs inside. 

Continuing to use the `redis-cli` we can connect to the server just like the SMB server in the previous example. Navigation through the server involves using a set of keywords.

![Redis Server](/assets/img/redeemer/flag.png)

We first select the database we want using `select 0`. `0` because the keyspace was listed under index 0 in the enumeration step. Next `keys *` lists all of the keys, and the value under the flag key is the one we want.

## Explosion

This box dealt with the Microsoft RDP service. I've seen this service a lot before, and it is usually a place where I will often get stuck on trying to bruteforce or just work around the credentials. With the right username and password through the use of`xfreerdp` you can gain an entire remote connection to a windows machine. Not CLI or terminal access but full GUI.

Either way this box starts with a scan

![NMAP Scan](/assets/img/explosion/scan.png)

Once again there are a multitude of open ports, many of which seem to be dealing with the same thing. In particular we focus on port 3389. Similarly to the other box there might be other services worth looking in to, however the box clearly pushes me towards Microsoft RPC, which has 3389 as its default port.

After that it is straight into `xfreerdp` and trying to gain access. I initially had a bit of a hang-up with a the command since everytime I had used it previously `xfreerdp` had been the command which was now not showing up as an executable. Trying to install the underlying package xfreerdp3 did nothing since it claimed the latest version was already installed. The fix, luckily, turned out to be pretty simple since the new command as just `xfreerdp3`. This must have happened recently as the last time remember using it the utility was just `xfreerdp`. 

Normally I would try some wordlists but the guiding questions made me think that there was some sort of default access. Thankfully `administrator` and a blank password let me connect:

![Connection](/assets/img/explosion/connect.png)

After that, the flag was just sitting in a NotePad file on the desktop.

![Flag](/assets/img/explosion/flag.png)

## Preignition

This box is our first foray into HTTP servers. Put more simply it is the first website we've been asked to hack.

To start we can show that this box is hosting a website in two ways. Firstly we can just type the IP into the browser which redirects us to a homepage. However not knowing this I did another NMAP scan:

![NMAP Scan](/assets/img/preignition/scan.png)

Only one open port is showing, a http service on the most common port for that, 80. After this I immediately verified that this was a website by going to the IP in the browser. 

The guiding questions were asking about `gobuster` a command line utility which I've only ever had experience using for directory busting/enumeration. The idea is that there are hidden URL's that are unlisted but are still able to be visited if you know the address manually. So, firing up GoBuster with a common wordlist we quickly get the results:

![Gobuster Scan](/assets/img/preignition/gobuster.png)

The only subdomain found was an `admin.php` page which gave the HTTP code 200 or "okay" upon visit. 

Going to the website `$ip/admin.php` we see:

![Login Page](/assets/img/preignition/login.png)

Again, normally I would try to bruteforce it, but with how simple these boxes are a couple of default tries and `admin` for both username and password presents us with the flag

![Flag](/assets/img/preignition/flag.png)

# Conclusion

This was a fun set of boxes, mostly because they were too easy to get stuck on in most places. Additionally the guided questions were fun to answer along the way, since it was motivating to see the green checkmarks. Overall it was good to do something simple like this so that I grew into the habit of taking screenshots and planning my steps for the report later.

I think in the future if I'm going to be doing multiple boxes (Next post will be another track I would think), I would want to write each report as it comes along rather than doing all of the boxes and waiting till the last moment. I should also work on chopping my screenshot up a bit since I notice myself going through multiple steps trying to describe what's in the picture where it could just be two pictures. Aditionally I'm still too lazy to fix some quality of life stuff with regards to writiing these reports. I'm still missing spellcheck, but more importantly a local preview of how things look on the website before I publish. This is already 900 lines long, and I'll be shocked if everything works on the first try. After all I'm writing everything by hand, as well as typing out each path to link each individual image.

Still, concerns aside, this was another successful blog post where I learned some new concepts. 

Ugh and I still need to remove the stupid "Fork-Me" banner.
