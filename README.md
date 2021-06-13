# Useful Snippets
Some personal notes, to cut down on time spent searching for this stuff! ;-)

## Apt
Couple of useful `apt` syntax examples.

```bash
sudo apt-get update
apt-cache showpkg <<package>>
```  
^- *check details/availability of an uninstalled package*

```bash
apt policy <<package>>
```
^- *summary info*

```bash
apt show <<package>>
```
^- *detailed info*

## Bash Prompt
The default Bash user prompt on most of my VMs seems set to display the entire path to your working directory. A few layers deep and you're at the end of the line before you actually start typing. I prefer colours on, and display only the top-level of the folder.  
`sudo nano ~/.bashrc`

```bash
# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
force_color_prompt=yes


if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\W\$ '
fi
```
^- *un-comment the `force_color_prompt` setting, and towards the end of each `PS1` line there is a lower-case "w" (full path) which I've changed to an upper-case "W" (only the current directory).*

## Browsers
Force "reader" mode in Firefox for site with malformed tags, and where the icon doesn't automatically appear. (Often sidesteps simple javascript blocking of adblock, etc.)
```
about:reader?url=
```
^- *paste full url at the end*

**LAN Based Web Services**  
To avoid being forced to prefix local (LAN) web servers with "http(s)" in Firefox, open `about:config` and search for `network.dns.localDomains`.

## CMD

Mapping a network drive including credentials.
```dos
net use T: \\networkShare\Test /u:domainname\username password
```

## DNS
Query an MX record (example):
```bash
dig +short google.com mx

30 alt2.aspmx.l.google.com.
20 alt1.aspmx.l.google.com.
10 aspmx.l.google.com.
40 alt3.aspmx.l.google.com.
50 alt4.aspmx.l.google.com.
```

Having established a mail server name, lookup its IP address:
```bash
nslookup mail99.example.com

Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   mail99.example.com
Address: xx.xx.xx.xx
Name:   mail99.example.com
Address: xxx:xxx:xxx:xxx::xx
```

## Excel

Date to day of week:  
`=text(CELL,"dddd")`  
^- *where CELL is a date value.*

Unhide All Rows:  
`ctrl+shift+9`  
^- *select entire sheet first...*

## Exchange Powershell

SMTP forwarding, step 1: (eg/ `externaldom.co.uk`)

The external domain name needs to be added first.

```powershell
New-RemoteDomain -Name externaldom -DomainName externaldom.co.uk

Get-RemoteDomain externaldom | Select DomainName, AutoForwardEnabled

Remove-RemoteDomain externaldom
```
* add an external domain
* confirm
* remove (if required)

SMTP forwarding, step 2:

Then forward mailboxes as required. Here mail is delivered **and** forwarded.

```powershell
Set-Mailbox -Identity "Freduardo.Debargo" -DeliverToMailboxAndForward $true -ForwardingSMTPAddress "freduardo.debargo@externaldom.co.uk"

Get-Mailbox "Freduardo.Debargo" | Format-List ForwardingSMTPAddress, DeliverToMailboxandForward
```
^- *enclosing in quotes appears to deal with special characters, like the apostrophe in `O'Connell`, etc.*

SMTP forwarding, step 3:

To cancel...

```powershell
set-mailbox -identity "Freduardo.Debargo" -DeliverToMailboxAndForward $false

set-mailbox -identity "Freduardo.Debargo" -DeliverToMailboxAndForward $false -ForwardingSmtpAddress $null -forwardingaddress $null

get-mailbox "Freduardo.Debargo" | format-list forwardingsmtpaddress,delivertomailboxandforward
```
* just forward the message, don't deliver to local mailbox
* cancel forwarding altogether
* confirm

## Linux Misc.

Check folder sizes from current location:
```bash
du -chd 1 | sort -h
```
^- *size of each folder (only 1 level deep) sorted by size, plus a final total.*

Simple recursive find from current location:
```bash
find . -name 'mail*'
```
^- *recursive by default.*

Checking for USB devices:
```bash
lsusb
```

List block storage devices, and query a specific device: (for its UUID, for example)
```bash
lsblk

sudo blkid /dev/sda1
```

Unattended upgrade config files:
```bash
/etc/apt/apt.conf.d/20auto-upgrades
/etc/apt/apt.conf.d/50unattended-upgrades
```
^- *consider removal of old kernals, whether to set an auto-reboot time, etc*

## NMAP

Quick scan for hosts on local subnet:
```bash
sudo nmap -sP -T4 192.168.1.0/24 | grep 'Nmap'
```

Same, but on a VirtualBox host, although still not reliable:
```bash
sudo nmap -sn 192.168.1.* --open | grep '('
```
^- *sn for a 'ping scan' and '--open' should indicate whether the host is up.*

Check if a specified port is open:
```bash
sudo nmap -sS -Pn -p 25 x.x.x.x
```
^- *in this example, is port 25 listening?*

Check for supported TLS version / ciphers:
```bash
sudo nmap -Pn -p 25 --script ssl-enum-ciphers x.x.x.x
```

## OpenSSL
Check certificate dates from a command line:
```bash
echo | openssl s_client -connect jonathancraddock.com:443 -servername jonathancraddock.com 2>/dev/null | openssl x509 -noout -dates
```

#### Generate a CSR
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout example.key -out example.csr
```
^- *req*uest for *new key* using *2048 bit RSA*, not encrypted, hence *no DES*. And *.key* is the PRIVATE key and *.csr* is the request. (Country code, "GB", or refer to: https://www.digicert.com/ssl-certificate-country-codes.htm )

Check a CSR
```bash
openssl req -in example.csr -noout -text
```

Read a TLS certificate:
```bash
openssl x509 -in certificate.crt -text -noout
```

View available ciphers:
```bash
openssl ciphers -s -tls1_2
```
^- *in this example, list only the ciphers available if TLS1.2 is negotiated.*

For an A+ rating (at the time of writing) on the https://globalsign.ssllabs.com/ SSL checker, using only TLS v1.2 and with no "weak" cipher suites, I found I could enable:

* ECDHE-ECDSA-AES256-GCM-SHA384:
* ECDHE-RSA-AES256-GCM-SHA384:
* DHE-RSA-AES256-GCM-SHA384:
* ECDHE-ECDSA-CHACHA20-POLY1305:
* ECDHE-RSA-CHACHA20-POLY1305:
* DHE-RSA-CHACHA20-POLY1305:
* ECDHE-ECDSA-AES128-GCM-SHA256:

Generate Ephemeral Diffie-Hellman (DHE) parameters:
```bash
openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
```
^- *This is going to take a long time... around 5 mins on my $5 Digital Ocean droplet. Assuming you're wanting to include this in your Apache config, see syntax below.*
```bash
SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"
```

## Powershell
See when an AD user last changed their password:
```powershell
get-aduser "firstname.lastname" -properties PasswordLastSet
```

When was client last seen by WSUS?
```powershell
Get-WsusComputer -ComputerUpdateStatus Any -NameIncludes 'lap-'  | Format-Table -autosize > \\some\path\wsus-seen.txt
```
^- *assumes the asset name includes 'lap-' and pipe it to a text file.*

## PuTTY
Confirming a server fingerprint from the (physical/virtual) console, prior to connecting with PuTTY - for example:
```bash
cd /etc/ssh
sudo ssh-keygen -E md5 -l -f ssh_host_rsa_key.pub
```
^- *Note the "-l -f" switches above are "Lima" and "Foxtrot", lowercase.*

## Regex Recipes
From a passage of text, find all words that end with "ly":
```regex
/[a-zA-Z]{3,99}ly\b/gm
```
^- *assumes the words consist of at least one letter + 'ly', and are less than 99 letters long*

In the example above I was looking to perform a find and replace in Notepad++. To perform a replace using the found string, enclose the regex in brackets. Eg/ `([a-zA-Z]{3,99}ly\b)` and use the notation `$1` in the replace field.

#### Related Notepad++ Regex
Find a word, at the beginning of a new line, followed by a space and an open-bracket:
```regex
^[a-zA-Z]* \(
```

To replace a word at the start of the line followed by space and and an open-bracket, with word ~~ open-bracket:

Find what: `(^[ a-zA-Z0-9\-\']*) \(`  
Replace with: `$1~~\(`

To replace "word, space, open-bracket, *some word categories*, close-bracket, space" with the same, followed by ~~ again:

Find what: `(^[ a-zA-Z0-9\.\~\-\'\&\(]*)\) `  
Replace with: `$1\)~~`  
^- *Note the "find what" string has a space character at the end.*

Thus, a text based dictionary definition such as:
```text
Abrogate (a.) Abrogated; abolished.
```
...becomes a more easily delimited format, regardless of line length, additional use of brackets, and so on:
```text
Abrogate~~(a.)~~Abrogated; abolished.
```

## Sed
Simple find and replace in a text file:
```bash
sed -i -e 's/find/replace/g' readme.md
```
^- *-i process the file **inline**, -e **expression** (and the spaces matter!), -s **substitute**, followed by the "**find**" and "**replace**" strings, g to indicate a **global** replace. The filename is specified at the end of the command.*

## SMTP / Telnet / OpenSSL
View SMTP header using Telnet:
```bash
telnet [ip/host] 25
...
quit
```

Query TLS using OpenSSL:
```bash
openssl s_client -connect mail99.example.com:25 -name yourfqdn.com -starttls smtp
```
^- note, the SMTP `EHLO` will default to "mail.example.com" unless you add a `-name` switch. I discovered this after spending some time attempting to diagnose blacklisting issues... and I don't think this is highlighted in many online examples. If you add a `-debug` switch you can observe this behaviour in the hex dump.  
https://www.openssl.org/docs/manmaster/man1/s_client.html

## TAR Compressed Backup of Folder
The syntax is tar, then switches, then destination file, then source folder:
```bash
tar -cvzf backup_file.tar.gz /folder/sub-folder
```
^- *Here you have **c** - create and **v** -verbose (might be annoying!) with **z** - compress the archive, and **f** - file.*

To extract, use the following command:
```bash
tar -xvf backup_file.tar.gz -C /folder/new-folder
```
^- *Note the upper-case -C that specifies the destination folder.*

## Windows 10
Diagnosing WiFi issues for a user, if you are able to remotely access a command line on their laptop:
```dos
netsh wlan show interfaces
```
^- *Quick means of confirming user's SSID, Signal and Receive/Transmit rates.*

Check WiFi password:
```dos
netsh wlan show profiles name="SSID" key=clear
```

## Xen Server
SSH to Xen server and make an ISO available in a new storage repository:
```bash
xe sr-create name-label="MyISORepository" type=iso device-config:location=/var/opt/iso/ device-config:legacy_mode=true content-type=iso
```
^- *having ssh'd to the server, downloaded an ISO (wget, etc, to for example, /var/opt/iso/), the command above will make it available as a new storage repository.*

## YouTube-DL

Links:
> YouTube-DL -> https://youtube-dl.org/  
> Github -> https://github.com/ytdl-org/youtube-dl/  
> ffmpeg -> https://ffmpeg.org/  
> ffmpeg builds -> https://www.gyan.dev/ffmpeg/builds/  

For standalone use, it's sufficient to create a folder and copy `youtube-dl.exe` and `ffmpeg.exe` into it. The following example copies the world's best song into the current folder.

```dos
youtube-dl -x --audio-format mp3 https://www.youtube.com/watch?v=lXgkuM2NhYI

Directory of C:\Users\xxx\Videos\youtube
01/05/2021  12:36    <DIR>          .
01/05/2021  12:36    <DIR>          ..
16/12/2019  06:16         3,257,497 David Bowie - Heroes (Official Video)-lXgkuM2NhYI.mp3
09/04/2021  10:16       112,424,960 ffmpeg.exe
01/05/2021  10:42         8,171,402 youtube-dl.exe
```

#### AdGuard Home

There is some sort of inconsistent behaviour of downloads whilst running a DNS sinkhole. The message below is typical.

```dos
[youtube] lXgkuM2NhYI: Downloading webpage
ERROR: unable to download video data: <urlopen error [WinError 10061] No connection could be made because the target machine actively refused it>
```

Disabling AdGuard for this client resolves the error, although in the query log nothing is shown as blocked.

```dos
[youtube] lXgkuM2NhYI: Downloading webpage
[download] Destination: David Bowie - Heroes (Official Video)-lXgkuM2NhYI.m4a
[download] 100% of 3.21MiB in 00:00
[ffmpeg] Correcting container in "David Bowie - Heroes (Official Video)-lXgkuM2NhYI.m4a"
[ffmpeg] Destination: David Bowie - Heroes (Official Video)-lXgkuM2NhYI.mp3
```

Footnote: appears to be due to a custom filtering rule and currently works ok with it disabled; possible delay in the filter log being updated? (1st May 2021)

```bash
# ||r*-aige.googlevideo.com^$important
```

#### Some syntax examples:

Download audio only and save as an MP3, download MP4 video in highest available quality.

```dos
youtube-dl -x --audio-format mp3 https://www.youtube.com/watch?v=***
youtube-dl https://www.youtube.com/watch?v=***
```

## Shortcuts

[CSR](#generate-a-csr)  
[YouTube-DL](#youtube-dl)
