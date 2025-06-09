

To begin the challenge, I was provided with SSH credentials to access the target machine.

**Command Used:**
`ssh ctf@10.10.255.227`

After entering the password, I successfully connected and got a terminal on the remote machine, as well as my first flag in the ssh login message:

![Pasted image 20250609120529.png](Pasted%20image%2020250609120529.png)

# Linux Basics Task

I am tasked to create a new user and create a file, so I create a user and switch to it:

	![Pasted image 20250609125321.png](Pasted%20image%2020250609125321.png)

Then I create a new folder and a text file:

![Pasted image 20250609125604.png](Pasted%20image%2020250609125604.png)

then I exit to go back to ctf and the flags:

![Pasted image 20250609125710.png](Pasted%20image%2020250609125710.png)

---

# CTF Tasks

First thing I do as I am back with the ctf user is run the command `ls -la` to find any hidden files.

![Pasted image 20250609121144.png](Pasted%20image%2020250609121144.png)

I check this txt file with `cat .f.txt` to find the next flag, which seems to be System Flag #4:
`{H1d3_1n_pl41n_s1gh7}`

Since I am given the information that one of the flags are named "find_flag.txt" I decide to look for any .txt files on the system with "flag" in its name.

I use the command `find / -type f -iname '*flag*.txt'` to *find*  in */* (the entire filesystem) *-type f* regular files that contain *flag* anywhere in the name, followed by *.txt*

Doing this gives me a lot of permission denied errors, so I also make sure to repeat the command while supressing error messages using *2>/dev/null*
With the command: `find / -type f -iname '*flag*.txt' 2>/dev/null`
This is what I find:

![Pasted image 20250609122628.png](Pasted%20image%2020250609122628.png)

On trying to concatenate the first file, I receive this message:

![Pasted image 20250609122718.png](Pasted%20image%2020250609122718.png)

This seems to be Other Flag #2, as well as a hint for another file: *files.zip*
The only password I know so far is the password for the ssh, which is "ctf"
I keep this in mind, and continue with the three other flag text files.
I use `cat` to read them in the same way as before.

They contain:
	Webpage Flag #7: {Fl4g_fl4g_fl4g}
	A hash: e0ZsNGcyX2ZsNGcyX2ZsNGcyfQ==
	System Flag #1: {F1nd_Fl4g_Fun}

The hash seems to be base64, with the `==` padding at the end.
I try to run command `base64 -d /var/www/html/flag/flag/flag2.txt`

![Pasted image 20250609123953.png](Pasted%20image%2020250609123953.png)

and receive what seems to be Webpage Flag #8

Earlier, I was given the info that there is a *files.zip* so I run the command ``find / -type f -iname 'files.zip' 2>/dev/null
Same as before, but specifically looking for one file which I have the full name of.
I try unzipping this file using the only password I know for now ``ctf

![Pasted image 20250609130059.png](Pasted%20image%2020250609130059.png)

It does not work, so I move on for now.

I start looking through directories instead:

![Pasted image 20250609131848.png](Pasted%20image%2020250609131848.png)

I see a story.txt, and read it.
It contained a bunch of 13375P34K, but I found System Flag #3 in there as well.

![Pasted image 20250609132030.png](Pasted%20image%2020250609132030.png)

I try anothern wide sweep of the file system with another command that narrows things down a bit.
``find / -type f -iname '*.txt' -exec grep -H '{' {} \; 2>/dev/null | less

As before, it looks from the root directory and down with ``find /`` for standard filetypes, but this time, any file of a *.txt* type. 
``-exec grep -H '{' {} \;
This part searches for a *{* character inside each file found, and prints the filename in place of the ``{}
the *-H* lets *grep* know to print the filename as well as the line that matches the *{* character found.
I make sure to supress error messages with ``2>/dev/null`` again
I use `| less` to be able to scroll through the results at my own pace.

the first page returns two flags I have already see, but also some new flags!

![Pasted image 20250609133552.png](Pasted%20image%2020250609133552.png)

{S3cr3t_Fl4g} and {Robots_Flag} appear to be Webpage Flag #3 and #5, respectively.

I immediately check to see if these files contain any more info as well before continuing the sweeping technique.

![Pasted image 20250609134028.png](Pasted%20image%2020250609134028.png)

It seems like the contained only the flags, and nothing else that would help me, so I continue looking a bit after new search results.

After looking a bit more with the same *find* command, I found nothing more, and it seems like there would be a waste of time to continue searching like this, so I try to find a new approach.

I try looking into the *flag* directory, but it seems that there are just loads of stacked subdirectories here, so I `cd ..` back to root and run a command to find all .txt files in *flag* and its subdirectories.

![Pasted image 20250609135725.png](Pasted%20image%2020250609135725.png)

reading the flag returns the last System Flag (#3), which contains a parentheses at the front rather than a brace.

Having found all the System Flags, I move my attention onto Webpage Flags


---


I first try to simply access the IP in my browser, and I am met with a flag on the landing page.

![Pasted image 20250609140629.png](Pasted%20image%2020250609140629.png)

I now try to look through the page source, to see if I find anything,

![Pasted image 20250609140727.png](Pasted%20image%2020250609140727.png)

![Pasted image 20250609152241.png](Pasted%20image%2020250609152241.png)

Here we find Webpage Flag #2, but I cannot find anything more of interest, so I will try to use gobuster to look for hidden files or directories.

``gobuster dir -u 10.10.238.132 -w /usr/share/wordlists/dirb/common.txt -x html,php,txt -t 40 -o gobuster_results.txt

this searched for the url given by *-u*, and uses the wordlist given with *-w*
I give it file extensions I want to find with *-x* (html, php, txt) and speed it up by using 40 threads with *-t 40*
Lastly, I output the info into gobuster_results.txt with *-o*

![Pasted image 20250609141627.png](Pasted%20image%2020250609141627.png)

I look through the results, and I look for specific names, filesizes or directories that might indicate a flag.
Some of these I have found already, such as robots and secret.txt, but  */hide.html* looks interesting, and also the *flag* directory has a 301 status code, meaning it is a permanent redirect, so I try accessing it in a browser first by visiting the IP address of the tryhackme machine, and appending /flag.

![Pasted image 20250609142205.png](Pasted%20image%2020250609142205.png)

This brings me to an Apache directory, and I enter the flag/ directory to find two .txt files.

![Pasted image 20250609142243.png](Pasted%20image%2020250609142243.png)

these are two of the flags (Webpage Flag #7 and #8) I have already found, unfortunately, so I move on.

I also try to access the */hide.html* found by gobuster, and am greeted by Webpage Flag #4

![Pasted image 20250609142545.png](Pasted%20image%2020250609142545.png)

We already found the first Webpage Flag via */index.html*, so let's try */index2.html*
At first this seems like a very genuine page, but there is a sneaky flag hidden here.

![Pasted image 20250609142903.png](Pasted%20image%2020250609142903.png)

``{C0nf1gur4t10n_Fl4g}`` is Webpage Flag #6

I am out of ideas for the Webpage Flags for now, so I move on to the Hash Flags

---


to work with the hash files on the attack box, I copy the folder recursively to the attack box with scp

![Pasted image 20250609145550.png](Pasted%20image%2020250609145550.png)

First up: hash1.txt

I install hashid, and run it to determine the hash type

![Pasted image 20250609145935.png](Pasted%20image%2020250609145935.png)

MD5 is quite common, so I will try with that one first.

![Pasted image 20250609150125.png](Pasted%20image%2020250609150125.png)

``C0d3_0b5cur3r_Flag`` is hash1.txt flag, so that worked.



for hash2.txt, SHA-1 seems likely.

![Pasted image 20250609150323.png](Pasted%20image%2020250609150323.png)

So I use John The Ripper, targeting SHA-1

![Pasted image 20250609150417.png](Pasted%20image%2020250609150417.png)

``C0d3_5l4y3r_Flag`` is hash2.txt flag.



hash3.txt seems to be SHA-512.

![Pasted image 20250609150534.png](Pasted%20image%2020250609150534.png)


![Pasted image 20250609150619.png](Pasted%20image%2020250609150619.png)

``H4ck3r_Flag`` is hash3.txt result.



next is is hash4.txt

![Pasted image 20250609150737.png](Pasted%20image%2020250609150737.png)

SHA-256 seems to be the most common one, and most likely here, so I try cracking that with John.

![Pasted image 20250609150850.png](Pasted%20image%2020250609150850.png)

``L0ck_Flag`` is hash4.txt flag.


for hash5.txt MD5 seems the most likely again.

![Pasted image 20250609151026.png](Pasted%20image%2020250609151026.png)


![Pasted image 20250609151040.png](Pasted%20image%2020250609151040.png)

``S3cur1ty_Flag`` is hash5.txt flag.

and that's all the hash flags.

Now all that is left is the *files.zip* and something called the "John flag"

I recall that the files.zip is at ``/srv/ftp/files.zip`` 
I scp it over to the attackbox so I can try to crack it.

![Pasted image 20250609153937.png](Pasted%20image%2020250609153937.png)

Now, I first try to extract the hash in order to maybe crack the password with john.

![Pasted image 20250609154040.png](Pasted%20image%2020250609154040.png)

After this, I try to crack it, using the rockyou.txt wordlist

![Pasted image 20250609154112.png](Pasted%20image%2020250609154112.png)

Unfortunately, as shown by the two ``0g`` indicators, it did not work.

At this point, I just start guessing a bit. I try unzipping and typing in the password. The hint at tryhackme tells me ``We have used it in the past multiple times``
I spend quite some time looking around at something I can use. I find that Webpage Flag #7 and #8 are quite similar, and wonder if this could perhaps be it.

![Pasted image 20250609154452.png](Pasted%20image%2020250609154452.png)

I first try unzipping the files.zip and typing the password in, this does not work. I run the unzip command with a -P password parameter so make sure it is correctly typed, and it is also wrong. This also reveals that there are two files inside the files.zip

![Pasted image 20250609154347.png](Pasted%20image%2020250609154347.png)

I think I spent about 30 minutes trying to look for some kind of pattern, but I couldn't find anything. After a while I noticed that the tryhackme hint also includes the format in which the answer needs to be given, which is a bunch of stars. 12 stars, to be exact.
``************``
I spent some more time trying to figure out whether I had missed some clue in the lab, something that amoounted to 12 symbols. I tried a bunch of options, including but not limited to:
	Fl4gfl4gfl4g
	FlagFlagFlag
	flagflagflag
Flags seemed to be the only recurring common denominator.
I was struggling to find something that "we have used *multiple* times in the past"

Then I took a couple minutes break, and when I sort of had given up, I eventually just realized "Masterschool" has 12 characters, and I tried it.

![Pasted image 20250609154800.png](Pasted%20image%2020250609154800.png)

I could not believe it actually worked, but imagine my relief. As I unzipped the files.zip, I also see that the secret.zip contains the "John flag" I was curious about earlier.

![Pasted image 20250609154901.png](Pasted%20image%2020250609154901.png)

To finally access the final flag, I use zip2john to get the zip hash

![Pasted image 20250609155003.png](Pasted%20image%2020250609155003.png)

Then I use the wordlist.txt included in the files.zip

![Pasted image 20250609155021.png](Pasted%20image%2020250609155021.png)

It returns a password ``CTF_TIME`` and I am finally ready to open the last zip, that contains the flag I have been looking for.

![Pasted image 20250609155145.png](Pasted%20image%2020250609155145.png)

``{LetMe1n123!@#}`` is the final flag, and with that, the CTF is done, and I am very satisfied.
I had a great time exploring, puzzling and stumbling my way to a bunch of flags.

Now it is time to do one last task, the Nmap report.


---
# Nmap Scan

I start my scan via a TryHackMe attackbox, scanning a machine connected to a room designated for Nmap learning, and I use the following command to scan:  
``nmap -n -sV -sC -T4 10.10.117.127``

This scan:
- ignores DNS resolution to speed things up `-n`
- detects the versions of services running on open ports `-sV`
- runs the default scripts with the Nmap Scripting Engine `-sC`
- speeds up the scan using a more aggressive template `-T4`

---
Here is the resulting output:


<details>
<summary>Click here to view full Nmap scan results</summary>



Host is up (0.00040s latency).  
Not shown: 995 filtered ports  
PORT STATE SERVICE VERSION  
21/tcp open ftp FileZilla ftpd  
| ftp-anon: Anonymous FTP login allowed (FTP code 230)  
|_Can't get directory listing: TIMEOUT  
| ftp-syst:  
|_ SYST: UNIX emulated by FileZilla  
53/tcp open domain?  
| fingerprint-strings:  
| DNSVersionBindReqTCP:  
| version  
|_ bind  
80/tcp open http Microsoft IIS httpd 10.0  
| http-methods:  
|_ Potentially risky methods: TRACE  
|_http-server-header: Microsoft-IIS/10.0  
|_http-title: IIS Windows Server  
135/tcp open msrpc Microsoft Windows RPC  
3389/tcp open ms-wbt-server Microsoft Terminal Services  
| rdp-ntlm-info:  
| Target_Name: WIN-SCAN  
| NetBIOS_Domain_Name: WIN-SCAN  
| NetBIOS_Computer_Name: WIN-SCAN  
| DNS_Domain_Name: win-scan  
| DNS_Computer_Name: win-scan  
| Product_Version: 10.0.17763  
|_ System_Time: 2025-06-09T16:03:26+00:00  
| ssl-cert: Subject: commonName=win-scan  
| Not valid before: 2025-06-08T15:54:36  
|_Not valid after: 2025-12-08T15:54:36  
|_ssl-date: 2025-06-09T16:03:56+00:00; -1s from scanner time.  
MAC Address: 02:E6:74:71:6D:E7 (Unknown)  
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

</details>


---

## Scan Results Overview

```
Host is up (0.00040s latency).  
Not shown: 995 filtered ports  
PORT STATE SERVICE VERSION  
21/tcp open ftp FileZilla ftpd  
53/tcp open domain?  
80/tcp open http Microsoft IIS httpd 10.0  
135/tcp open msrpc Microsoft Windows RPC  
3389/tcp open ms-wbt-server Microsoft Terminal Services``
```

The host is up and responded with very low latency (0.00040s).  
Nmap found **5 open TCP ports**, with most others marked as filtered. These are the results:

| Port | Service       | Version Info                | Notes                                                      |
| ---- | ------------- | --------------------------- | ---------------------------------------------------------- |
| 21   | FTP           | FileZilla ftpd              | Allows anonymous login — potential security risk           |
| 53   | DNS           | Unknown                     | Responds to version queries, but service wasn't identified |
| 80   | HTTP          | Microsoft IIS httpd 10.0    | TRACE method is enabled — could allow XST attacks          |
| 135  | Microsoft RPC | Microsoft Windows RPC       | Standard Windows RPC service                               |
| 3389 | RDP           | Microsoft Terminal Services | Windows RDP exposed; system appears to be Win 10           |

---

## Potential vulnerabilities

###  FTP (Port 21)
```
21/tcp open ftp FileZilla ftpd  
| ftp-anon: Anonymous FTP login allowed (FTP code 230)  
|_Can't get directory listing: TIMEOUT  
| ftp-syst:  
|_ SYST: UNIX emulated by FileZilla``
```

Nmap reports that *anonymous login is enabled* on the FTP service. This can be a big risk if sensitive files are accessible without authentication.  
I wasn’t able to retrieve a directory listing due to a timeout, but in general, *anonymous FTP should be disabled* unless there’s a specific reason to allow it.

Suggestion:  
- Disable anonymous access in the FileZilla server config and restrict login to known users.

---

### HTTP (Port 80)
```
80/tcp open http Microsoft IIS httpd 10.0  
| http-methods:  
|_ Potentially risky methods: TRACE  
|_http-server-header: Microsoft-IIS/10.0  
|_http-title: IIS Windows Server``
```

The HTTP service is running on *Microsoft IIS 10.0*, and Nmap notes that the *TRACE method is enabled*.

Further research tells me TRACE isn’t usually needed, and having it active could allow for *Cross Site Tracing (XST)* attacks, where an attacker could potentially steal cookies or other headers.

Suggestion: 
- Disable TRACE in IIS using the ``web.config`` file or via ``URLScan.ini``

---

### RDP (Port 3389)
```
3389/tcp open ms-wbt-server Microsoft Terminal Services  
| rdp-ntlm-info:  
| Target_Name: WIN-SCAN  
| NetBIOS_Domain_Name: WIN-SCAN  
| NetBIOS_Computer_Name: WIN-SCAN  
| DNS_Domain_Name: win-scan  
| DNS_Computer_Name: win-scan  
| Product_Version: 10.0.17763``
```

RDP is open and shows the machine is named `WIN-SCAN`, running Windows version 10.0.17763.

While this is expected in a CTF, in a real scenario, leaving RDP open to the internet is risky, and is a common target for brute-force attacks and exploits.

Suggestion:  
- Limit RDP to internal access or VPN, enable Network Level Authentication, and enforce strong password policies.

---

### DNS (Port 53)
```
53/tcp open domain?  
| fingerprint-strings:  
| DNSVersionBindReqTCP:  
| version  
|_ bind``
```

This port is open and does respond, but the version detection was inconclusive. The server replied to a DNS version query with "bind", which could suggest it’s using BIND (Berkeley Internet Name Domain) or something similar, but it wasn’t fully recognized.

Suggestion: 
- Make sure the DNS service isn’t leaking version info externally, and limit recursive queries to trusted clients only.

---

## RDP SSL Certificate

```
| ssl-cert: Subject: commonName=win-scan
| Not valid before: 2025-06-08T15:54:36
|_Not valid after:  2025-12-08T15:54:36
```


The RDP service uses an SSL certificate for `win-scan`. It was issued just a day before the scan, so it’s pretty fresh. 
Since there’s no sign it’s from a trusted authority, it’s probably self-signed. 

This might be common in labs like this, but in a real-world environment, it would be better to use a certificate issued by a trusted authority.

---

## Summary

This scan gave me a good look at the surface of the target. The FTP and HTTP services stood out as potentially vulnerable. If this were a real-world engagement, these would be the first areas I’d dig into further or recommend locking down.

This task was a helpful exercise in recognizing risks just by looking at open ports and service banners.
```
