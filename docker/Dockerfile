FROM debian:latest

EXPOSE 137/udp 138/udp
EXPOSE 139 445

RUN apt update 
RUN apt install -qq -y samba
RUN apt install -qq -y samba-common-bin
RUN apt install -qq -y lsof
RUN apt install -qq -y vim
RUN apt install -qq -y nano
RUN apt clean 
RUN rm -rf /var/lib/apt/lists/*

WORKDIR /app
