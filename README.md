# CentOS 7.7 Minimal Installation as Offline Standalone Server on Raspberry Pi 4

Raspberry Pi is a cheap mini PC with very high specs. Besides being able to be used as a desktop PC, Raspberry Pi can also be used as an application server. In offline applications, the server requires a timekeeper that can keep time running even if the server loses power. Unfortunately, Raspberry Pi is not equipped with a real time clock on the board. For that, we need to add our own real time clock module.

The addition of real time clock requires a real time clock module or RTC and software that will synchronize the time between the Raspberry Pi and the RTC module.

## Download Image

The following mirrors should have the ISO images available:

1. http://mirror.0x.sg/centos-altarch/7.7.1908/isos/armhfp/
2. http://mirror.aktkn.sg/centos-altarch/7.7.1908/isos/armhfp/
3. http://mirror.nsw.coloau.com.au/centos-altarch/7.7.1908/isos/armhfp/
4. http://mirror.xtom.com.hk/centos-altarch/7.7.1908/isos/armhfp/
5. http://mirror-hk.koddos.net/centos-altarch/7.7.1908/isos/armhfp/
6. http://mirrors.huaweicloud.com/centos-altarch/7.7.1908/isos/armhfp/
7. http://ftp.yz.yamagata-u.ac.jp/pub/linux/centos-altarch/7.7.1908/isos/armhfp/
8. http://mirror.genesishosting.com/centos-altarch/7.7.1908/isos/armhfp/
9. http://centos-altarch.itsbrasil.net/7.7.1908/isos/armhfp/
10. http://mirror.math.princeton.edu/pub/centos-altarch/7.7.1908/isos/armhfp/
11. http://centos-altarch.reloumirrors.net/7.7.1908/isos/armhfp/
12. http://mirrors.ocf.berkeley.edu/centos-altarch/7.7.1908/isos/armhfp/
13. http://mirror.ehv.weppel.nl/centos-altarch/7.7.1908/isos/armhfp/
14. http://mirror.dal.nexril.net/centos-altarch/7.7.1908/isos/armhfp/
15. http://mirrors.powernet.com.ru/centos-altarch/7.7.1908/isos/armhfp/
16. http://mirrors.coreix.net/centos-altarch/7.7.1908/isos/armhfp/
17. http://linux.darkpenguin.net/distros/CentOS-AltArch/7.7.1908/isos/armhfp/
18. http://centos-altarch.mirror.liquidtelecom.com/7.7.1908/isos/armhfp/
19. http://ftp.osuosl.org/pub/centos-altarch/7.7.1908/isos/armhfp/
20. http://mirror.atl.genesisadaptive.com/centos-altarch/7.7.1908/isos/armhfp/
21. http://mirrors.dotsrc.org/centos-altarch/7.7.1908/isos/armhfp/
22. http://mirrors.xtom.com/centos-altarch/7.7.1908/isos/armhfp/
23. http://mirror.airenetworks.es/CentOS-AltArch/7.7.1908/isos/armhfp/
24. http://ftp.agdsn.de/pub/mirrors/centos-altarch/7.7.1908/isos/armhfp/
25. http://mirror.ufs.ac.za/centos-altarch/7.7.1908/isos/armhfp/
26. http://quantum-mirror.hu/mirrors/pub/centos-altarch/7.7.1908/isos/armhfp/
27. http://mirrors.uniri.hr/centos-altarch/7.7.1908/isos/armhfp/
28. http://mirror.yer.az/CentOS-AltArch/7.7.1908/isos/armhfp/
29. http://mirror.vcu.edu/pub/gnu_linux/centos-altarch/7.7.1908/isos/armhfp/
30. http://ftp.bme.hu/centos-altarch/7.7.1908/isos/armhfp/

## Update YUM


The Yellowdog Updater, Modified (YUM) is a free and open-source command-line package-management utility for computers running the Linux operating system using the RPM Package Manager. Though YUM has a command-line interface, several other tools provide graphical user interfaces to YUM functionality.

YUM allows for automatic updates and package and dependency management on RPM-based distributions. Like the Advanced Package Tool (APT) from Debian, YUM works with software repositories (collections of packages), which can be accessed locally or over a network connection.

This step is optional. You can skip this if we will not update YUM.

```bash
yum update -y
```

## Install Tools

Nano is text editor on Linux to create or modify text file.
Sysstat is tool to show the CPU status using command line.
ZIP is tool to zip files and directories.
Unzip is tool to extract zipped file.

```bash
yum install -y nano
yum install -y sysstat
yum install -y zip
yum install -y unzip
```

## Install RTC DS3231

The DS3231 is a low-cost, extremely accurate I2C real-time clock (RTC) with an integrated temperature-compensated crystal oscillator (TCXO) and crystal. The device incorporates a battery input, and maintains accurate timekeeping when main power to the device is interrupted. The integration of the crystal resonator enhances the long-term accuracy of the device as well as reduces the piece-part count in a manufacturing line. 

The DS3231 is available in commercial and industrial temperature ranges, and is offered in a 16-pin, 300-mil SO package.The RTC maintains seconds, minutes, hours, day, date, month, and year information. The date at the end of the month is automatically adjusted for months with fewer than 31 days, including corrections for leap year. The clock operates in either the 24-hour or 12-hour format with an AM/PM indicator. Two programmable time-of-day alarms and a programmable square-wave output are provided. Address and data are transferred serially through an I2C bidirectional bus.

We need to install i2c-tools to read real time clock from the i2c device. After install i2c-tools, we need to create a service to synchronize the real time clock from RTC DS3231 to Raspberry Pi when system is started.

```bash
echo -e "dtparam=i2c_arm=on" >> /boot/config.txt
echo -e "dtoverlay=i2c-rtc,ds3231" >> /boot/config.txt
echo -e "i2c-dev" >> /etc/modules-load.d/i2c.conf
yum -y install i2c-tools
i2cdetect -y 1
echo -e '[Unit]' > /usr/lib/systemd/system/rtc.service
echo -e 'Description=rtc' >> /usr/lib/systemd/system/rtc.service
echo -e '' >> /usr/lib/systemd/system/rtc.service
echo -e '[Service]' >> /usr/lib/systemd/system/rtc.service
echo -e 'ExecStart=/sbin/hwclock -s' >> /usr/lib/systemd/system/rtc.service
echo -e '' >> /usr/lib/systemd/system/rtc.service
echo -e '[Install]' >> /usr/lib/systemd/system/rtc.service
echo -e 'WantedBy=multi-user.target' >> /usr/lib/systemd/system/rtc.service
systemctl enable rtc.service
systemctl start rtc.service
```

## Install Access Point with DHCP 

```bash
yum install -y dhcp
echo -e 'ESSID=PicoEdu' > /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'MODE=Ap' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'KEY_MGMT=WPA-PSK' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'MAC_ADDRESS_RANDOMIZATION=default' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'TYPE=Wireless' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'PROXY_METHOD=none' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'BROWSER_ONLY=no' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'BOOTPROTO=none' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPADDR=192.168.0.11' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'PREFIX=24' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'GATEWAY=192.168.0.1' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'DNS1=8.8.8.8' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'DEFROUTE=yes' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPV4_FAILURE_FATAL=no' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPV6INIT=yes' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPV6_AUTOCONF=yes' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPV6_DEFROUTE=yes' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPV6_FAILURE_FATAL=no' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'IPV6_ADDR_GEN_MODE=stable-privacy' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'NAME=wlan0' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'UUID=605a8783-c38b-4351-8f28-e82f99fdd0c6' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'ONBOOT=yes' >> /etc/sysconfig/network-scripts/ifcfg-wlan0
echo -e 'WPA_PSK=planetbiru' > /etc/sysconfig/network-scripts/keys-wlan0
echo '' > /etc/dhcp/dhcpd.conf
echo 'default-lease-time 600;' >> /etc/dhcp/dhcpd.conf
echo 'max-lease-time 7200;' >> /etc/dhcp/dhcpd.conf
echo 'authoritative;' >> /etc/dhcp/dhcpd.conf
echo '' >> /etc/dhcp/dhcpd.conf
echo 'subnet 192.168.0.0 netmask 255.255.255.0 {' >> /etc/dhcp/dhcpd.conf
echo '    option routers                  192.168.0.1;' >> /etc/dhcp/dhcpd.conf
echo '    option subnet-mask              255.255.255.0;' >> /etc/dhcp/dhcpd.conf
echo '    option broadcast-address        192.168.0.255;' >> /etc/dhcp/dhcpd.conf
echo '    range 192.168.0.40 192.168.0.254;' >> /etc/dhcp/dhcpd.conf
echo '}' >> /etc/dhcp/dhcpd.conf
echo -e '[Unit]' > /usr/lib/systemd/system/dhcp.service
echo -e 'Description=DHCPv4 Server Daemon' >> /usr/lib/systemd/system/dhcp.service
echo -e 'Documentation=man:dhcpd(8) man:dhcpd.conf(5)' >> /usr/lib/systemd/system/dhcp.service
echo -e 'Wants=network-online.target' >> /usr/lib/systemd/system/dhcp.service
echo -e 'After=network-online.target' >> /usr/lib/systemd/system/dhcp.service
echo -e 'After=time-sync.target' >> /usr/lib/systemd/system/dhcp.service
echo -e '' >> /usr/lib/systemd/system/dhcp.service
echo -e '[Service]' >> /usr/lib/systemd/system/dhcp.service
echo -e 'Type=notify' >> /usr/lib/systemd/system/dhcp.service
echo -e 'ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid wlan0' >> /usr/lib/systemd/system/dhcp.service
echo -e '' >> /usr/lib/systemd/system/dhcp.service
echo -e '[Install]' >> /usr/lib/systemd/system/dhcp.service
echo -e 'WantedBy=multi-user.target' >> /usr/lib/systemd/system/dhcp.service
systemctl enable dhcpd.service
systemctl start dhcpd.service
```

## Install Apahce - PHP - MariaDB

```bash
firewall-cmd --permanent --add-port=800/tcp
firewall-cmd --permanent --add-port=80/tcp
yum install -y httpd
yum install -y mariadb-server mariadb
yum install -y php php-gd php-gd php-exif php-libxml php-mbstring php-mysql
systemctl enable httpd.service
systemctl enable mariadb.service
chown -R apache /var/www/html
chmod -R 755 /var/www/html
systemctl start mariadb.service
systemctl start httpd.service
```
