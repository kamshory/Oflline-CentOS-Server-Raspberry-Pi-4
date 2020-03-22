# CentOS 7.7 Minimal Installation as Offline Standalone Server on Raspberry Pi 4

## Update YUM


The Yellowdog Updater, Modified (YUM) is a free and open-source command-line package-management utility for computers running the Linux operating system using the RPM Package Manager. Though YUM has a command-line interface, several other tools provide graphical user interfaces to YUM functionality.

YUM allows for automatic updates and package and dependency management on RPM-based distributions. Like the Advanced Package Tool (APT) from Debian, YUM works with software repositories (collections of packages), which can be accessed locally or over a network connection.

This step is optional. You can skip this if you will not update YUM.

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
