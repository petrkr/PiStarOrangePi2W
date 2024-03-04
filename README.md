# Manual install Pi-Star on Orange Pi Zero 2W

- [Manual install Pi-Star on Orange Pi Zero 2W](#manual-install-pi-star-on-orange-pi-zero-2w)
  - [Install required packages](#install-required-packages)
  - [Turn on 32bit armhf, because prebuilt binaries are compiled like that](#turn-on-32bit-armhf-because-prebuilt-binaries-are-compiled-like-that)
  - [Turn off UART0 serial console](#turn-off-uart0-serial-console)
  - [Turn on I2C](#turn-on-i2c)
  - [Create user and grand permissions](#create-user-and-grand-permissions)
  - [Install mmdvm binaries](#install-mmdvm-binaries)
  - [Install pi star scripts](#install-pi-star-scripts)
  - [Install web](#install-web)
  - [Setup up web servers](#setup-up-web-servers)
  - [Copy pistar files](#copy-pistar-files)
    - [usr local](#usr-local)
    - [Now ugly part comes](#now-ugly-part-comes)

## Install required packages
```shell
apt install git php-fpm php-json php-mbstring php-zip nginx stm32flash shellinabox geoip-database zip avahi-daemon avrdude flashrom
```

## Turn on 32bit armhf, because prebuilt binaries are compiled like that
TODO: Probably create debian packages or podman images with pre-built images for arm64 arch too, also redone update strategy from git checkout to system packages as that will solve also dependencies and architecture

```shell
dpkg --add-architecture armhf
apt install libc6:armhf libusb-1.0-0:armhf libgps28:armhf
```


## Turn off UART0 serial console
Yes, because all MMDVM HATs using UART0 rather than other UARTs, we have to use UART0. That is not mistake by design, but really first versions of Rpi had only two UARTs and probably noone want solve switching to UART1.

`orangepi-config`  
System --> Bootenv  
replace `"console=both"` for `"console=none"` and add `"extraargs=console=tty1"`

```shell
systemctl stop serial-getty@ttyS0
systemctl disable serial-getty@ttyS0
```

## Turn on I2C
System --> Hardware --> enable pi-i2c1, in system it will appear as /dev/i2c-2


## Create user and grand permissions
`adduser mmdvm`
 - check UID 1001 and home /home/mmdvm

`usermod -a -G i2c,dialout mmdvm`
 - TODO spi group, which not exists

## Install mmdvm binaries
TODO: out of dated, binaries are NOT compatible with arm64 and armhf wiringpi is not available to support all MMDVMHost
```shell
cd /usr/local/
git clone --depth 1 https://github.com/AndyTaylorTweet/Pi-Star_v4_Binaries_Bin.git bin
```

## Install pi star scripts
```shell
cd /usr/local
git clone --depth 1 https://github.com/AndyTaylorTweet/Pi-Star_Binaries_sbin.git sbin
```

## Install web
```shell
cd /var/www/
git clone --depth 1 https://github.com/AndyTaylorTweet/Pi-Star_DV_Dash.git dashboard
```

## Setup up web servers
configure nginx  
TODO: copy site-available files  
TODO: copy nginx default.d files  
Note: change PHP FPM socket acording to PHP version  
Create/copy .htpasswd to /var/www/.htpasswd and give www-data owner

## Copy pistar files

### usr local
/usr/local/etc
chown

### Now ugly part comes
Copy /etc pi-star specific files and make them owned by www-data, so web server can write to them :'(

- TODO file list / automatic something
- pistar-release
- mmdvmhost

```shell
chown www-data dstar-radio.mmdvmhost dstarrepeater hostname hosts ircddbgateway mobilegps pistar-css.ini pistar-remote starnetserver timeserver
```

 *I am not happy web browser have access to hosts file, it allow attacker to redirect IP/DNS to his server*


Set up sudo without password to web server (yes, I am not kidding but PHP is 
really full of sudo commands)

`visudo` and add following line at bottom `www-data ALL=(ALL) NOPASSWD: ALL`
