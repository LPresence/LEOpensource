# LEOpensource
Tp open source by Lukas Presencia

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
> NTP service: inactive


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

DNS over TLS est un protocole de sécurité permettant de chiffrer et d'encapsuler les requêtes DNS avec le protocole TLS. L'objectif est d'améliorer la confidentialité et la sécurité des utilisateurs en empêchant l'écoute et la manipulation des données DNS via un MITM.

#### Configuration du DNS over TLS 

> cat /etc/systemd/resolved.conf

           [Resolve]
           
           DNS=9.9.9.9
           
           FallbackDNS=1.1.1.1 8.8.8.8

           DNSOverTLS=opportunistic

Et la commande `resolvectl status`nous donne :
>          
           Global

           LLMNR setting: yes
       
           MulticastDNS setting: yes
           
           DNSOverTLS setting: opportunistic
           
           DNSSEC setting: allow-downgrade
           
           DNSSEC supported: yes
           
           Current DNS Server: 9.9.9.9
           
           DNS Servers: 9.9.9.9
           
           Fallback DNS Servers: 1.1.1.1
           
                                 8.8.8.8

Lors d'une requete `host google.fr`nous pouvons vérifier que le dns utilise le port 853 et donc passe en TLS :

> tcpdump -vvAs0 port 853

Cette commande affiche la requete dns et montre bien qu'elle passe par le port 853 (sécurisé donc). La meme commande sur le port 53 ne montre rien étant donné que le dns passe désormais par TLS sur le port 853.



## Logind

## Services

La commande `systemctl list-units`nous permet d'observer le processus chronyd et son unité :
> chronyd.service                                                                          loaded active running   NTP client/server

## Boot et Logs

La commande `systemd-analyze plot > graphe.svg` nous permet de génerer un grapique de la séquence de boot :

![alt text](https://github.com/LPresence/LEOpensource/blob/master/graphe.svg "graphe séquence boot 1")

## Cgroups

La commande `ps -e -o pid,cmd,cgroup` affiche les differents processus accompagnés de leurs cgroups. On peut voir que le cgroup utilisé par notre session ssh est `0::/user.slice/user-0.slice/session-4.scope`.

> cat /sys/fs/cgroup/user.slice/memory.max

           max

> systemctl set-property user.slice MemoryMax=512M

> cat /sys/fs/cgroup/user.slice/memory.max
          
          536870912
          
> ls -la /etc/systemd/system.control/user.slice.d/
           total 4
           drwxr-xr-x 2 root root  31 29 nov.  15:39 .
           
           drwxr-xr-x 3 root root  26 29 nov.  15:39 ..
           
           -rw-r--r-- 1 root root 149 29 nov.  15:39 50-MemoryMax.conf
           
### Dbus

> dbus-monitor --system      
           
           signal time=1575039116.000557 sender=:1.2 -> destination=(null destination) serial=1177 path=/org/freedesktop/systemd1/unit/chronyd_2eservice; interface=org.freedesktop.DBus.Properties; member=PropertiesChanged

           string "org.freedesktop.systemd1.Service"
           
Cet evenement intervient lors du redémarrage du service chronyd. L'objet /org/freedesktop/systemd1/unit/chronyd_2eservice emet un signal "PropertiesChanged" en broadcast vers l'interface org.freedesktop.DBus.Properties afin de modifier les propriétes us processus. 

Losqu'on demande le rafraichissement d'un autre service comme sshd, on s'aperçoit que les actions sont sensiblement les même avec une différence notable pour le champs `path=/org/freedesktop/systemd1/unit/sshd_2eservice`.

## Namespaces

Le cgroup utilisé lors du lancement d'un processus avec systemd-run sur ma machine est `─session-1.scope`.

La commande `systemd-run  -p MemoryMax=512M  --wait -t /bin/bash` permet d'executer un shell de facon sandboxée et avec une limite de mémoire ram de 512Mo.

Lors de l'execution d'un processus avec le parametre `-p IPAccounting=true ` on observe des informations supplémentaires a la sortie du processus :
>         
           Main processes terminated with: code=exited/status=0
           Service runtime: 43.798s
           CPU time consumed: 224ms
           IP traffic received: 588B
           IP traffic sent: 588B

Avec l'ajout des parametres ` -p IPAddressAllow=10.0.2.0/24 -p IPAddressDeny=any` on s'apercoit que le processus ne peut pas pinguer des machines hors du scope 10.0.2.0/24 :
>
           
           PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
           
           ping: sendmsg: Operation not permitted
           
           ping: sendmsg: Operation not permitted
           

La commande systemd-nspawn semble faire crash mon terminal. 

Avec l'argument `ephemeral ` le conteneur est exécuté avec un instantané temporaire de son file systeme qui est supprimé immédiatement lorsque le conteneur se termine. L'argument `private-network ` déconnecte la mise en réseau du conteneur. Cela rend toutes les interfaces réseau indisponibles dans le conteneur, à l'exception du périphérique loopback.

Il est par exemple possible de restreindre encore plus son execution avec des paramètres comme `--rlimit` pour limiter ses ressources.

## Skip to creation de services simples

