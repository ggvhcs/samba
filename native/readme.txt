# --- Samba share folder Without Docker on debian 12 labs productions ... --- #

--- github repo ---
https://github.com/ggvhcs/samba

Enviroment:
---
Debian 12.
---

--- installation. ---
$ dpkg -l samba
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

$ mkdir -p /srv/SHARE/ShareFolder
$ chown nobody:nogroup /srv/SHARE/ShareFolder
$ chmod 777 /srv/SHARE/ShareFolder

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

smbclient --version
smbclient -L localhost -U%
smbclient //localhost/ShareDirectory -Uadministrator -c 'ls'

--- bibliography ---
administración de redes gnu/linux (F. Código Libre Dominicano) Antonio Perpinan
