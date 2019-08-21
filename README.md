# Realtek R8101 linux driver source

This is the official linux driver source for Realtek RTL8101E/RTL8102E/RTL8103E/RTL8105E/RTL8106E/RTL8107E FE 100M NICs, obtained from [here](https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software). Tested on Ubuntu 19.04 in my Dell Inspiron 1440 laptop, works flawlessly.

Since Ubuntu 18.x these NICs are being used with the default r8169 driver (which is for the R8169 gigabit ethernet controller only), which is causing terrible download and upload speeds. Installing the official driver for 810xE series fixes it.

**Driver version**: 1.034  
**Release date**: 21/06/2019

How to install this driver?
----
* First of all, clone this repo to some folder and cd into that folder
* Install basic dependencies for compiling the driver: `sudo apt install build-essential linux-headers-$(uname -r)`
* Unload the existing r8169 driver: `sudo modprobe -r r8169`
* Block the r8169 driver in modprobe: `sudo sh -c 'echo blacklist r8169 >> /etc/modprobe.d/blacklist.conf'`
* Run the automatic build & install script: `sudo ./autorun.sh`. Ignore errors if any.
* Check if the driver is loaded: `lsmod | grep r8101`, it will return a single line like `r8101                 204800  0`
* If the driver is not loaded, then run `sudo modprobe r8101`.
* Now try connecting. If you don't see the connection then you might need a reboot.
* Enjoy ethernet speeds upto 100Mbps!

Some additional info
----
```
<Set the network related information>
	1. Set manually
		a. Set the IP address of your machine.

			# ifconfig ethX "the IP address of your machine"

		b. Set the IP address of DNS.

		   Insert the following configuration in /etc/resolv.conf.

			nameserver "the IP address of DNS"

		c. Set the IP address of gateway.

			# route add default gw "the IP address of gateway"

	2. Set by doing configurations in /etc/sysconfig/network-scripts
	   /ifcfg-ethX for Redhat and Fedora, or /etc/sysconfig/network
	   /ifcfg-ethX for SuSE. There are two examples to set network
	   configurations.

		a. Fixed IP address:
			DEVICE=eth0
			BOOTPROTO=static
			ONBOOT=yes
			TYPE=ethernet
			NETMASK=255.255.255.0
			IPADDR=192.168.1.1
			GATEWAY=192.168.1.254
			BROADCAST=192.168.1.255

		b. DHCP:
			DEVICE=eth0
			BOOTPROTO=dhcp
			ONBOOT=yes

<Modify the MAC address>
	There are two ways to modify the MAC address of the NIC.
	1. Use ifconfig:

		# ifconfig ethX hw ether YY:YY:YY:YY:YY:YY

	   ,where X is the device number assigned by Linux kernel, and
		  YY:YY:YY:YY:YY:YY is the MAC address assigned by the user.

	2. Use ip:

		# ip link set ethX address YY:YY:YY:YY:YY:YY

	   ,where X is the device number assigned by Linux kernel, and
		  YY:YY:YY:YY:YY:YY is the MAC address assigned by the user.

<Force Link Status>

	1. Force the link status when insert the driver.

	   If the user is in the path ~/r8101, the link status can be forced
	   to one of the 4 modes as following command.

		# insmod ./src/r8101.ko speed=SPEED_MODE duplex=DUPLEX_MODE autoneg=NWAY_OPTION

		where
			SPEED_MODE	= 100	for 100Mbps
					= 10	for 10Mbps
			DUPLEX_MODE	= 0	for half-duplex
					= 1	for full-duplex
			NWAY_OPTION	= 0	for auto-negotiation off (true force)
					= 1	for auto-negotiation on (nway force)
		For example:

			# insmod ./src/r8101.ko speed=100 duplex=0 autoneg=1

		will force PHY to operate in 100Mpbs Half-duplex(nway force).

	2. Force the link status by using ethtool.
		a. Insert the driver first.
		b. Make sure that ethtool exists in /sbin.
		c. Force the link status as the following command.

			# ethtool -s ethX speed SPEED_MODE duplex DUPLEX_MODE autoneg NWAY_OPTION

			where
				SPEED_MODE	= 100	for 100Mbps
						= 10	for 10Mbps
				DUPLEX_MODE	= half	for half-duplex
						= full	for full-duplex
				NWAY_OPTION	= off	for auto-negotiation off (true force)
						= on	for auto-negotiation on (nway force)

		For example:

			# ethtool -s eth0 speed 100 duplex full autoneg on

		will force PHY to operate in 100Mpbs Full-duplex(nway force).

<Jumbo Frame>
	RTL8101E, RTL8102E and RTL8103E do not support Jumbo Frame.
```
