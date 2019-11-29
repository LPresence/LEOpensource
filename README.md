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
>               Local time: ven. 2019-11-29 11:06:51 CET
>           Universal time: ven. 2019-11-29 10:06:51 UTC
>                 RTC time: ven. 2019-11-29 10:04:22
>                Time zone: Europe/Paris (CET, +0100)



