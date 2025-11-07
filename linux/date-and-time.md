# Date and Time

Enable NTP:

```sh
timedatectl set-ntp true
```

For `systemd`-installed distro: `/etc/systemd/timesyncd.conf`

```conf
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
```
