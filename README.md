# LEOpensource
Tp open source

## 1
`systemctl --version`
> systemd 243 (v243-4.gitef67743.fc31)


La commande `systemctl status` nous montre que systemd opère bien en PID1
> │ └─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 30
           
La commande `ps --ppid 2 -p 2 --deselect` permet de lister les processus système not kernel. Elle retourne une trentaine de processus comme `firewalld` qui gère le pare-feu, `sshd` qui gère le serveur ssh, `cockpit-ws` qui gère l'interface web de manageent du serveur ou encore `bash`ou `containerd`qui gère respectivement le shell bash et les conteneurs docker.

## 2

> [admin@localhost root]$ timedatectl

               Local time: ven. 2019-11-29 11:06:51 CET

           Universal time: ven. 2019-11-29 10:06:51 UTC

                 RTC time: ven. 2019-11-29 10:04:22

                Time zone: Europe/Paris (CET, +0100)
                
                System clock synchronized: no
                
              NTP service: active




Le Local Time correspond à l'heure de la machine sur son fuseau horaire, le Universal Time correspond à l'heure  "universelle" UTC 0 bien connue et le RTC Time correspond à l'horloge matérielle interne à la machine qui conserve l'heure même lorsqu'il est éteint. Cette horloge est utile pour suivre les modifications des dossier de manière très precise.

`timedatectl set-timezone Europe/Paris` pour passer sur le fuseau horaire français.

`timedatectl set-ntp 0` permet de desactiver le ntp.
>               NTP service: inactive


## 3 

Changer le hostname avec `hostnamectl set-hostname fedo1`.

Le nom d’hôte "pretty" est de haut niveau et peut inclure toutes sortes de caractères spéciaux, le nom d’hôte static est utilisé pour initialiser le nom d’hôte du noyau au démarrage et le nom d’hôte transient qui est une valeur de secours reçue de la configuration du réseau.

Le nom d'hote static est a privilégier en production.

## 4

### Networkmanager

> nmcli con show

           NAME     UUID                                  TYPE      DEVICE
 
           enp0s3   ca39e4fd-ceda-3fee-9d51-8dd4b9828c6b  ethernet  enp0s3
 
           docker0  c658a840-70e9-4af1-b8b9-6b5c29275fc8  bridge    docker0
 
 
> [root@fedo1 ~]#  cat /var/lib/NetworkManager/internal-ca39e4fd-ceda-3fee-9d51-8dd4b9828c6b-enp0s3.lease

           This is private data. Do not parse.

           ADDRESS=10.0.2.10

           NETMASK=255.255.255.0

           ROUTER=10.0.2.1

           SERVER_ADDRESS=10.0.2.3

           T1=600

           T2=1050

           LIFETIME=1200

           DNS=10.33.10.20 10.33.10.2 8.8.8.8 8.8.4.4

           DOMAINNAME=auvence.co

           CLIENTID=010800274ab3b3

Pour stopper NetworkManager : `systemctl stop NetworkManager`

### Systemd-networkd

Pour activer systemd-networkd : `systemctl start systemd-networkd`

Configuration de /etc/systemd/network/enp0s3network :

> vim /etc/systemd/network/enp0s3network

           [Match]

           Key=enp0s3

           [Network]
           
           Address=10.0.2.10/24
           
           GATEWAY=10.0.2.1
           
           DNS=10.33.10.20



### Systemd-resolv

`systemctl start systemd-resolved && systemctl enable systemd-resolved`

La commande `netstat -laputn`nous montre un serveur dns local :

> udp        0      0 127.0.0.53:53           0.0.0.0:*                           1143/systemd-resolv

