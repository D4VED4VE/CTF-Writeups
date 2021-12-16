# Pickle Rick - THM CTF Writeup
## Introduction
This is an easy level CTF from Try Hack Me requiring us to exploit a webserver and find 3 'ingredients' (flags) to turn rick back into a human from a pickle :D
- Tags: ctf, dirbuster, linux
<br>

### Scanning for open ports and services using nmap
We can run an nmap scan to see which ports are open and what is running on the web application
```bash
nmap -sC -sV -oN nmap/initial IP_ADDRESS
```
- -sC for standard scripts
- -sV to enumerate versions
- -oN to output to a file as well as the terminal
	- Output saved to initial in directory nmap

![alt text](https://github.com/dave250783/CTF-Writups/TryHackMe/Pickle_Rick/pickle-rick-web-page.png?raw=true)

As we can see from the scan there are only 2 common ports open;
- 22 SSH
	- OpenSSH 7.2p2 Ubuntu 4ubuntu2.6
	- We need login credentials before we can do anything with this
- 80 http
	- Apache/2.4.18 (Ubuntu)
	- We can go and look at the website to get started
<br>

## Looking around the website
We can now investigate the website running on port 80 and see if we can find any information.

![pickle-rick-web-page.png](:/e44d215ac5724a9fb7064514f7551379)

From the homepage we can gather some information. We need to;
- Login to rick's computer
- Find three secret ingredients - the flags we need 
- Find Rick's password
<br>

Looking at the source of the homepage we see a comment from rick

![pickle-rick-web-source.png](:/f30780f852484d13999637797db94801)

- We now have a username to use so we only need to find a password for the account
<br>

If we keep looking around we can find the password. So we can see if there is a robots.txt file on this webserver.
- A lot of webservers have this file to instruct search engine crawlers what not to publically index in their search results
- We find a rick and morty reference in the robots.txt file. This be the password that goes with our found username?
<br>

There doesnt seem to be any more information on the website so we can move on to the next stage.
<br>

## Finding available directories with gobuster
We can also enumerate the directories on the webserver using gobuster. We get a hint that this will be required by the tags given to the ctf.
```bash
gobuster dir -u IP_ADDRESS -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,sh, txt,cgi,html,css,js,py
```
- Use dir to enumerate directories
- Here I used the dirbuster 2.3 medium list as the wordlist
- I added the extensions php, sh, txt, cgi, html, css, js, py using the -x switch
	- This will go through our list with each of the extensions to see if these files are present

![pickle-rick-gobuster.png](:/a58085a1ee8744099f2f388b4b91d873)

From the gobuster results we see that we have only found three directories;
- /login.php
	- Status: 200 - This returned ok
	- This looks like a good place to start
- /assets
	- Status: 301 - This is a redirect
	- This redirects us to /assets/
	- We find nothing of interest here
- /server-status
	- Status: 403 - We do not have permission to access this directory
<br>

## Logging into the website and accessing the server
### Logging in
The reference we found in the robots.txt file was the password for our username we found earlier. 

After logging into the website at the /login.php location we are faced with a 'command panel'.

### Accessing the server
We know that the box is running Ubuntu linux from our nmap scan so we can try some linux commands.
Inputting 'ls' shows us the files in our current directory which has a file containing our first ingredient
![pickle-rick-web-command-panel.png](:/043fadf70d774109bda445ced7e40219)

We can try using 'cat' to display the contents of the file but it says the command is blocked. We can instead use the 'less' command to open the file and display the first flag.

## Finding the second ingredient on the filesystem
We see another file called clue.txt and this file tells us to look around the file system for another ingredient.

I used the find command to find the next ingredient as i thought it was more efficient than bumbling around the file system blind.
```bash
find / -type f -name '*ingredient*'
```
- I simply searched in the root of the file system recursively for a file with 'ingredient' in the name

![pickle-rick-web-second-ingredient.png](:/ee165aa955c74891922422518adb98f0)

- We have now found the second ingredient and can print it out using the less command enter it as our flag
```bash
less /home/rick/second\ ingredients
```
- Remember to escape the space character in the file name

## Finding the thrid ingredient
There were no clues after i got the second ingredient so i tried looking around the filesystem  for the ingredient and then later i remembered we had SSH so tried to look for an rsa key but no luck in either case.

It then occured to me that i hadnt checked if i had sudo privelages.
```bash
sudo -l
```
- This returned that i could do anything as sudo using no password
	- I felt a little silly at this point for forgetting to check but meh lol
- I ran ls on the root directory and sure enough there was the third ingredient
```bash
sudo ls /root
```
![pickle-rick-web-third-ingredient.png](:/9d7f4eade3ce498a944e1b8625143cea)

- I ran the less command on the file using sudo to get the last flag and enter it.
