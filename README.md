
# CentOS 7.7 Minimal Installation as Offline Standalone Server on Raspberry Pi 4

Raspberry Pi is a single board computer with very high specs. Besides being able to be used as a desktop PC, Raspberry Pi can also be used as an application server. In offline applications, the server requires a timekeeper that can keep time running even if the server loses power. Unfortunately, Raspberry Pi is not equipped with a real time clock on the board. For that, we need to add our own real time clock module.

The addition of real time clock requires a real time clock module or RTC and software that will synchronize the time between the Raspberry Pi and the RTC module.

## Download Image

To get mirror links of the ISO images, visit

http://isoredirect.centos.org/altarch/7/isos/armhfp/

You can use newer version of CentOS

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

### Tools Installation Commands

```bash
yum install -y nano
yum install -y sysstat
yum install -y zip
yum install -y unzip
```

## Install RTC DS3231

The DS3231 is a low-cost, extremely accurate I2C real-time clock (RTC) with an integrated temperature-compensated crystal oscillator (TCXO) and crystal. The device incorporates a battery input, and maintains accurate timekeeping when main power to the device is interrupted. The integration of the crystal resonator enhances the long-term accuracy of the device as well as reduces the piece-part count in a manufacturing line. 

The DS3231 is available in commercial and industrial temperature ranges, and is offered in a 16-pin, 300-mil SO package.The RTC maintains seconds, minutes, hours, day, date, month, and year information. The date at the end of the month is automatically adjusted for months with fewer than 31 days, including corrections for leap year. The clock operates in either the 24-hour or 12-hour format with an AM/PM indicator. Two programmable time-of-day alarms and a programmable square-wave output are provided. Address and data are transferred serially through an I2C bidirectional bus.

To synchronize time between RTC module and Raspberry Pi, we use hwclock. hwclock is a utility for accessing the hardware clock, also referred to as the Real Time Clock (RTC). The hardware clock is independent of the operating system you use and works even when the machine is shut down. This utility is used for displaying the time from the hardware clock. hwclock also contains facilities for compensating for systematic drift in the hardware clock.

The hardware clock stores the values of: year, month, day, hour, minute, and second. It is not able to store the time standard, local time or Coordinated Universal Time (UTC), nor set the Daylight Saving Time (DST). 

We need add ```dtparam=i2c_arm=on``` and ```dtoverlay=i2c-rtc,ds3231``` to ```/boot/config.txt```, add ```i2c-dev``` to ```/etc/modules-load.d/i2c.conf``` and create a service to execute ```hwclock -s``` when system is started.

**Syntax:**

```hwclock [function] [option...]```

**Functions:**
-   **-r, --show :** It will display the RTC time.
-   **--get :** It is used to display the drift corrected RTC time.
-   **--set :** It is used to set the RTC according to –date.
-   **-s, --hctosys :** It will set the system time form the RTC.
-   **-w, --systohoc :** It will set the RTC from the system time i.e. the opposite of _-s, –hctosys_.
-   **--systz :** Used to send the timescale configurations to the kernel.
-   **-a, --adjust :** Adjust the RTC to account for systematic drift.
-   **--predict :** It will predict the drifted RTC time according to –date.

**Options:**

-   **-u, --utc :** Shows that RTC timescale is UTC.
-   **-l, --localtime :** Shows that RTC timescale is Local.
-   **-D, --debug :** This is used to display the debug information. Information is shown according to the demands of hwclock command.
-   **-V, --version :** Display version information and exit.
-   **-h, --help :** Display help text and exit.

### RTC Installation Commands

```bash
echo -e "dtparam=i2c_arm=on" >> /boot/config.txt
echo -e "dtoverlay=i2c-rtc,ds3231" >> /boot/config.txt
echo -e "i2c-dev" >> /etc/modules-load.d/i2c.conf
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

The Dynamic Host Configuration Protocol (DHCP) is a network management protocol used on Internet Protocol networks whereby a DHCP server dynamically assigns an IP address and other network configuration parameters to each device on a network so they can communicate with other IP networks. A DHCP server enables computers to request IP addresses and networking parameters automatically from the Internet service provider (ISP), reducing the need for a network administrator or a user to manually assign IP addresses to all network devices. In the absence of a DHCP server, a computer or other device on the network needs to be manually assigned an IP address, or to assign itself an APIPA address, which will not enable it to communicate outside its local subnet.

DHCP can be implemented on networks ranging in size from home networks to large campus networks and regional Internet service provider networks. A router or a residential gateway can be enabled to act as a DHCP server. Most residential network routers receive a globally unique IP address within the ISP network. Within a local network, a DHCP server assigns a local IP address to each device connected to the network. 

### Access Point with DHCP Installation Commands

```bash
yum install -y dhcp
echo -e 'ESSID=PlanetPOS' > /etc/sysconfig/network-scripts/ifcfg-wlan0
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

## Setup Ethernet

### Ethernet Setup Commands

```bash
echo -e 'TYPE="Ethernet"' > /etc/sysconfig/ifcfg-eth0
echo -e 'BOOTPROTO=none' >> /etc/sysconfig/ifcfg-eth0
echo -e 'NM_CONTROLLED="yes"' >> /etc/sysconfig/ifcfg-eth0
echo -e 'DEFROUTE="yes"' >> /etc/sysconfig/ifcfg-eth0
echo -e 'NAME="eth0"' >> /etc/sysconfig/ifcfg-eth0
echo -e 'UUID="a5ae9a6c-3951-4e8a-b99d-a4ea5dc33bf1"' >> /etc/sysconfig/ifcfg-eth0
echo -e 'ONBOOT="yes"' >> /etc/sysconfig/ifcfg-eth0
echo -e 'DNS1=8.8.8.8' >> /etc/sysconfig/ifcfg-eth0
echo -e 'IPV4_FAILURE_FATAL=no' >> /etc/sysconfig/ifcfg-eth0
echo -e 'IPV6INIT=no' >> /etc/sysconfig/ifcfg-eth0
echo -e 'IPADDR=192.168.0.11' >> /etc/sysconfig/ifcfg-eth0
echo -e 'PREFIX=24' >> /etc/sysconfig/ifcfg-eth0
echo -e 'GATEWAY=192.168.0.1' >> /etc/sysconfig/ifcfg-eth0
echo -e 'NETMASK=255.255.255.0' >> /etc/sysconfig/ifcfg-eth0
echo -e 'DNS2=8.8.4.4' >> /etc/sysconfig/ifcfg-eth0 
```

## Install Apahce - PHP - MariaDB

If our applications are written in PHP and using MariaDB database, we need to install Apache Web Server, PHP Runtime and MariaDB Database Server.

### Server Installation Commands

```bash
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
