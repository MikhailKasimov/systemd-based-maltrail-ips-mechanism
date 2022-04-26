# systemd-based-maltrail-ips-mechanism

systemd-realization for Maltrail IPS-mechanism

## Introduction

Maltrail server's configuration [maltrail.conf](https://github.com/stamparm/maltrail/blob/master/maltrail.conf) section `[Server]` has the option, which allows to forward information, collected by Maltrail's sensor(-s), to ```ipset/iptables``` for blocking purposes:

Option `FAIL2BAN_REGEX` contains the regular expression (e.g. `attacker|reputation|potential[^"]*(web scan|directory traversal|injection|remote code)|spammer|mass scanner`) to be used in `/fail2ban` web calls for extraction of today's attacker source IPs. This allows the usage of IP blocking mechanisms (e.g. `fail2ban`, `iptables` or `ipset`) by periodic pulling of blacklisted IP addresses from remote location. Example usage would be the following script (e.g. run as a `root` cronjob on a minute basis):

```sh
#!/bin/bash
ipset -q flush maltrail
ipset -q create maltrail hash:net
for ip in $(curl http://127.0.0.1:8338/fail2ban 2>/dev/null | grep -P '^[0-9.]+$'); do ipset add maltrail $ip; done
iptables -I INPUT -m set --match-set maltrail src -j DROP
```

Save this script in Maltrail's work directory ( e.g. ```/opt/maltrail/maltrail-ips.sh```) and make it executable by ```chmod +x /opt/maltrail/maltrail-ips.sh``` command.

- There're two variants of periodical running ```/opt/maltrail/maltrail-ips.sh```

1) ```crontab-way``` on a minute basis:

1a) Open bash-shell terminal and do ```sudo crontab -eu root``` command.

2a) Put ```* * * * * root /opt/maltrail/maltrail-ips.sh``` and save current configuration for ```crontab``` file.

2) ```systemd-way``` on a minute basis:

2a) Download and put ```/maltrail-ips.timer``` and ```maltrail-ips.service``` to ```/etc/systemd/system/``` directory.

2b) Do ```sudo systemctl daemon-reload``` command to force systemd service manager to re-read its own configuration.

2c) Run ```sudo systemctl start maltrail-ips.timer``` and ```sudo systemctl enable maltrail-ips.timer``` commands to have ```/maltrail-ips.timer``` able to automatically start after system gets booted.

## Checks

a) Run ```curl http://127.0.0.1:8338/fail2ban``` command to get list of addresses, which will be forwarded to ```iptables``` to block.

b) Run ```systemctl status maltrail-ips.timer``` and ```systemctl status maltrail-ips.service```:

![photo_2022-04-25_14-10-45](https://user-images.githubusercontent.com/7167300/165260489-b950a24d-7788-48bc-a517-a9029f4a9021.jpg)

c) To track systemd logs for ```/maltrail-ips.timer``` and ```/maltrail-ips.service``` run:

```sudo journalctl -f -u maltrail-ips.timer``` and ```sudo journalctl -f -u  maltrail-ips.service``` commands.

d) Run ```sudo iptables -L -n -v``` to check ```iptables``` statistics:

![photo_2022-04-25_14-23-26](https://user-images.githubusercontent.com/7167300/165260182-f5570845-8742-4b70-8cf3-d527a4a3c02f.jpg)

## License

This software is provided under a MIT License as the original [Maltrail project](https://github.com/stamparm/maltrail/blob/master/README.md#license). See the accompanying [LICENSE](https://github.com/stamparm/maltrail/blob/master/LICENSE) file for more information.

## Links:

[Maltrail Project](https://github.com/stamparm/maltrail)

[Maltrail Wiki](https://github.com/stamparm/maltrail/wiki)
