## Comandos do timedatectl

```
timedatectl  status
timedatectl
timedatectl | grep Time

timedatectl list-timezones
timedatectl list-timezones |  egrep  -o "Asia/B.*"
timedatectl list-timezones |  egrep  -o "Europe/L.*"
timedatectl list-timezones |  egrep  -o "America/N.*"

timedatectl set-timezone "Asia/Kolkata"
timedatectl set-timezone UTC

timedatectl set-time 15:58:30
timedatectl set-time 20151120
timedatectl set-time '2015-11-20 16:14:50'

timedatectl | grep local

timedatectl set-local-rtc 1
timedatectl set-local-rtc 0

timedatectl set-ntp true
timedatectl set-ntp false

timedatectl set-ntp true
timedatectl set-timezone Europe/Berlin

```

