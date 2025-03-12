# --- Samba share folder Without Docker on debian 12 labs productions ... --- #

--- github repo ---
https://github.com/ggvhcs/samba

--- youtube ---
https://youtu.be/KwI1sr6_hus

Enviroment:
---
Ubuntu 22.04.
---

--- installation. ---
$ apt-get update //update repos db.
$ apt-get -y install samba smbclient //install samba and samba client.

$ cp /etc/samba/smb.conf /etc/samba/smb.conf.default // create an copy of default conf file.

--- create users samba ---
$ groupadd smbusers
$ adduser --no-create-hom --shell /usr/sbin/nologin smbuser
$ smbpasswd -a smbuser
$ usermod -G smbusers smbuser
$ groups smbuser

--- create share samba folders ---
$ mkdir -p /srv/SHARE/ShareDirectory
$ chown nobody:nogroup /srv/SHARE/ShareDirectory
$ chmod 777 /srv/SHARE/ShareDirectory

--- another share Folder ---
$ mkdir -p /srv/SHARE/ShareFolder
$ chown nobody:nogroup /srv/SHARE/ShareFolder
$ chmod 777 /srv/SHARE/ShareFolder

--- !we need make some changes on smb.conf ---

--- restart and check the service ---
$ systemctl restart smbd nmbd

testparm

--- check the port is up ---
lsof -Pni :137
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nmbd    3169 root   13u  IPv4  53357      0t0  UDP *:137 
nmbd    3169 root   15u  IPv4  53369      0t0  UDP 192.168.35.7:137 
nmbd    3169 root   16u  IPv4  53370      0t0  UDP 192.168.35.255:137

lsof -Pni :445
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
smbd    3180 root   44u  IPv6  53480      0t0  TCP *:445 (LISTEN)
smbd    3180 root   46u  IPv4  53482      0t0  TCP *:445 (LISTEN)

smbclient --version
smbclient -L localhost -U%
smbclient //localhost/ShareDirectory -Uadministrator -c 'ls'

--- bibliography ---
administración de redes gnu/linux (F. Código Libre Dominicano) Antonio Perpinan
