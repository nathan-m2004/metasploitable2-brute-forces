# metasploitable2-brute-forces

brute forcing a few of the metasploitable 2 services

# nmap scan

first we scan the server ip to see wich services are listening, you can see the log in `nmap_scan.txt`
here are the services we want to brute force:

```
PORT     STATE SERVICE      VERSION
21/tcp   open  ftp          vsftpd 2.3.4
22/tcp   open  ssh          OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
80/tcp   open  http         Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp  open  netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```
