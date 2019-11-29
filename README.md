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

Changer le hostname avec `hostnamectl set-hostname fedo1`
Le nom d’hôte "pretty" est de haut niveau et peut inclure toutes sortes de caractères spéciaux, le nom d’hôte static est utilisé pour initialiser le nom d’hôte du noyau au démarrage et le nom d’hôte transient qui est une valeur de secours reçue de la configuration du réseau.

Le nom d'hote static est a privilégier en production.

## 4



