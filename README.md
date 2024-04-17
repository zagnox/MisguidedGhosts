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

9. Tentojme XSS dhe SSTI me burp intruder per te gjetur vulnerabilitete. Me burpsuite arrijme te dallojme nje xss qe eshte patchuar. Tentojme te shmangim filtrat dhe arrijme te gjejm nje payload qe funksionon.
10. Hapi i rradhes eshte te kapim cookien e adminit duke derguar nje request drejt IP tone.
11. Pasi kapim cookien dhe logohemi kryejme directory brute forcing dhe zbulojme /photos. Kur ngarkojme dicka shohim qe ne url kemi nje parameter te quajtur image. Duke tentuar per directory traversal dhe command injection disa komanda jane suksesshme.
12. Kur tentojme te marrim nje reverse shell me command injection shohim se hapesirat filtrohen. Per te shmangur hapesirat perdorim ${IFS}.
13. Pasi morem shellin gjetem disa file me instruksion dhe nje rsa key ne direkotrine e Zac. Celesi rsa per ssh eshte i koduar me cipher. Per te gjetur shifren e cipherit. Pas nje kerkimi ne google kuptojme qe celesat RSA fillojne ne shumicen e rasteve me 3 karaktere MII. Duke ditur kete informacion ne mund te tentojme te bejme brute force deri sa te gjejme cipherin qe perputhet me karakteret e celesit tone.
14. Pasi kemi dekoduar celesin ne mund te hyjme me ssh si zac. Hapi tjeter eshte te eskalojme privilegjet ne root
15. Pasi kryejme kerkim automartik me linpeas shohim qe portat 22 139 dhe 445 ne localhost jane hapur dhe smb lejon logim si anonim. Perodrim ligolo-ng per te bere forward gjithe portat lokale drejt attackboxit tone.
16. Ne smb gjejme nje file me passworde te cilin e perodrim per te bere brute force ne ssh userit hayley. Pasi gjejme passwordin logohemi me ssh.
17. Serish duke perdorur linpeas shohim se nje session i tmux mund te behet hijack si root. Shkruajme komanden dhe kemi nje tmux session si root.


