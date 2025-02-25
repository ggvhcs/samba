# --- *** Samba share folder with Docker in Linux Mint 21. *** --- #

--- github repo ---
https://github.com/ggvhcs/samba

Develop Enviroment:
---
Linux Mint 21.2 Mate x64.
Docker version 24.0.7, build 24.0.7-0ubuntu2~22.04.1
git version 2.34.1
Github Desktop version 3.2.0-linux1 (x64)
Visual Studio Code version 1.96.4
---

1 --- download image debian latest from hub docker ---
$ sudo docker pull debian:latest
$ sudo docker images |grep debian


$ cd ~/Documents/GitHub/linux/samba
$ ls -l
---
   394 Feb 22 13:21 Dockerfile
   777 Feb 22 13:39 native-samba.txt
  3255 Feb 25 08:19 readme.txt
    43 Feb 22 13:21 run-samba.sh
 4096 Feb 25 07:07 ShareDirectory
 4096 Feb 25 07:05 ShareFolder
  1037 Feb 25 07:06 smb.conf
---

$ chmod 777 -Rvf ../samba
$ chown nobody:nogroup -Rvf ../samba

2 --- Create the Dockerfile file --- #
$ touch Dockerfile
$ nano Dockerfile
$ cat Dockerfile 
---
FROM debian:latest

EXPOSE 137/udp 138/udp
EXPOSE 139 445

RUN apt update -qq && apt install -qq -y samba
RUN apt install -qq -y samba-common-bin
RUN apt install -qq -y lsof
RUN apt install -qq -y vim
RUN apt install -qq -y nano
RUN apt clean 
RUN rm -rf /var/lib/apt/lists/*

WORKDIR /app

ADD run-samba.sh /app/run-samba.sh
RUN chmod +x /app/run-samba.sh

CMD [ "bash", "/app/run-samba.sh" ]
---

# --- Create the run-samba.sh file --- #
$ touch run-samba.sh
$ cat run-samba.sh
---
/usr/sbin/nmbd && /usr/sbin/smbd
---

--- create image from debian latest ---
$ sudo docker build -t debian_samba .
$ sudo docker images |grep samba
---
debian_samba                     latest           57691f3b30f7   30 hours ago    468MB
---

--- create an network bridge ---
$ sudo docker network create --subnet=172.15.0.0/16 homenet
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
5d945100191c   homenet   bridge    local

$ sudo docker run -ti --name smbdriver \
--net homenet --ip 172.15.0.20 \
-d --name dockersmb \
-v $(pwd):/app \
-v $(pwd)/smb.conf:/etc/samba/smb.conf \
--interactive --tty --entrypoint /bin/bash debian_samba

$ sudo docker ps

--- entramos al container ---
$ sudo docker exec -it b75d45f92a5b bash

--- version --
$ samba --version
---
Version 4.17.12-Debian
---

--- create users samba ---
$ groupadd smbusers
$ adduser --no-create-hom --shell /usr/sbin/nologin smbuser
$ smbpasswd -a smbuser
$ usermod -G smbusers smbuser
$ groups smbuser

--- create share samba folders ---
$ mkdir -p /app/ShareDirectory
$ chown nobody:nogroup /app/ShareDirectory
$ chmod 777 /app/ShareDirectory

$ mkdir -p /app/ShareFolder
$ chown nobody:nogroup /app/ShareFolder
$ chmod 777 /app/ShareFolder

$ --- run the service --- #
$ /usr/sbin/nmbd
$ /usr/sbin/smbd

--- Test the setting in smb.conf ---
testparm

lsof -Pni :137
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nmbd    6085 root   16u  IPv4  25746      0t0  UDP *:137 
nmbd    6085 root   18u  IPv4  25749      0t0  UDP 10.10.2.9:137 
nmbd    6085 root   19u  IPv4  25750      0t0  UDP 10.10.2.127:137 

lsof -Pni :445
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
smbd    6137 root   34u  IPv6  24820      0t0  TCP *:445 (LISTEN)
smbd    6137 root   36u  IPv4  24822      0t0  TCP *:445 (LISTEN)

telnet 172.15.0.20 445

# Auto Start Containers after System Reboot.

$ cd ~
$ sudo nano /etc/systemd/system/docker-samba.service

---
[Unit]
Description=dockersmb
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start dockersmb
ExecStop=/usr/bin/docker stop dockersmb

[Install]
WantedBy=default.target
---

sudo systemctl enable docker-samba.service
sudo systemctl disable docker-samba.service

# --- *** Samba share folder Native in Linux Mint 21. *** --- #

