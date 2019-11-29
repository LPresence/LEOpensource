# LEOpensource
Tp open source

`systemctl --version`
> systemd 243 (v243-4.gitef67743.fc31)


La commande `systemctl status` nous montre que systemd opère bien en PID1
> │ └─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 30
           
la commande `ps --ppid 2 -p 2 --deselect` permet de lister les processus système not kernel

