# UniFi Build
The following is for creating a UniFi network. These instructions will setup your `LAN Networks`, `Wireless Networks` and `Routing & Firewall`.

You will need a UniFi Controller WebGUI to perform the tasks.

The hardware in this recipe uses:
>  *  1x Ubiquiti UniFi Security Gateway 3P (USG)
>  *  1x Ubiquiti UniFi Switch 24
>  *  1x Ubiquiti UniFi Switch 8 POE-150W
>  *  1x Ubiquiti UniFi AP-AC-Pro
>  *  1x Synology DS1515+ with 4x NICs; and,
>  *  Various hardware for a Proxmox cluster (Qotom Mini PC Q500G6-S05).

Modify these instructions to meet your own hardware requirements.

Network prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.0.0.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or goes down)
- [x] A DDNS service is fully configured and enabled (I recommend you use the free Synology DDNS service)
- [x] A ExpressVPN account (or any preferred VPN provider) is valid and its smart DNS feature is working (public IP registration is working with your DDNS provider)

Other Prerequisites are:
- [x] Installation of UniFi Controller on your PC or notebook (later to be moved to a Proxmox UniFi LXC)

Tasks to be performed are:
- [1.00 UniFi Controller](#100-unifi-controller)
- [2.00 UniFi Settings - Site](#200-unifi-settings---site)
- [3.00 UniFi Settings - Wireless Networks](#300-unifi-settings---wireless-networks)
- [2.0 UniFi Networks](#20-unifi-networks)
	- [2.1 Edit Default LAN network](#21-edit-default-lan-network)
	- [2.2 Create UniFi Networks](#22-create-unifi-networks)
- [3.00 UniFi Routing & Firewall](#300-unifi-routing--firewall)
- [4.00 UniFi IPS](#400-unifi-ips)
- [5.00 UniFi DPI](#500-unifi-dpi)
- [6.00 UniFi Guest Control](#600-unifi-guest-control)
- [7.00 UniFi Profiles](#700-unifi-profiles)
- [8.00 UniFi Services](#800-unifi-services)
	- [8.01 UniFi Services - DHCP](#801-unifi-services---dhcp)
	- [8.02 UniFi Services - MDNS](#802-unifi-services---mdns)
	- [8.03 UniFi Services - UPNP](#803-unifi-services---upnp)
	- [8.04 UniFi Services - NTP](#804-unifi-services---ntp)
	- [8.05 UniFi Services - Scheduled Upgrades](#805-unifi-services---scheduled-upgrades)
- [9.00 UniFi Admins](#900-unifi-admins)
- [10.00 UniFi User Groups](#1000-unifi-user-groups)
- [11.00 UniFi Controller](#1100-unifi-controller)
- [12.00 UniFi User Interface](#1200-unifi-user-interface)
- [12.00 UniFi Notifications](#1200-unifi-notifications)
- [13.00 UniFi Cloud Access](#1300-unifi-cloud-access)
- [14.00 UniFi Elite Device](#1400-unifi-elite-device)
- [15.00 UniFi Maintenance](#1500-unifi-maintenance)
- [16.00 UniFi Auto Backup](#1600-unifi-auto-backup)
- [00.00 Patches & Fixes](#0000-patches--fixes)


## 1.00 UniFi Controller
To be completed.

## 2.00 UniFi Settings - Site
Here we set you basic UniFi site (i.e home, location of site etc) preferences.  Go to your UniFi controller `Settings` > `Site` and set the values as follows, remembering to click `Apply Changes` when done:

| Site | Value | Notes
| :--- | :--- | :---
| **Site Configuration**
| Site Name | site1.foo.bar | *Use a identifier name like `berlin house` or `berlin.foo.bar`*
| Country | Select your nation
| Timezone| Select your Timezone
| **Services**
| Advanced Features | `☐` Enable advanced features
| Automatic Upgrades | `☑`  Automatically upgrade AP firmware
| LED | `☑`  Enable status LED | *If disturbing LEDs, disable*
| Alerts | `☐` Enable alert emails
| Speed Test | `☐` Enable periodic speed test everyminutes
| Uplink Connectivity Monitor | `☐` Enable connectivity monitor and wireless uplink
| Remote Logging | `☐` Enable remote Syslog server
| | `☐` Enable Netconsole logging server
| **Provider Capabilities**
| Download | `500` mbps | *Set according to your ISP WAN provider DL subscription*
| Upload | `500` mbps | *Set according to your ISP WAN provider UL subscription*
| **Auto-Optimize Network**
| Automatically Optimize Network and Wi-Fi performance | `Off`
| **Device Authentication**
| SSH Authentication | `☑` Enable SSH authentication
| | Username `admin`
| | Password | `Type Passward` | *Random character password ONLY i.e cA(8&KxjLHz8s4?A). And note it down as its vital for recovery of AP's.*
| SSH Keys | No SSH keys have been defined.

And click `Apply Changes`.

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_site.png)

## 3.00 UniFi Settings - Wireless Networks
Here we create wireless networks in VLAN increments of 10 (i.e VLAN10, VLAN20) which correspond to our `Network` settings.

Go to your UniFi controller `Settings` > `Wireless Networks` > `+Create New Wireless Network` and set the values as follows, remembering to click `Apply Changes` when done:

| Wireless Networks| Value | Notes
| :--- | :--- | :---
| **Create New Wireless Network**
| Name/SSID | `name-open` | *Choose your own SSID such as `banana-open`*
| Enabled | `☑` Enable this wireless network
| Security |`☐` Open `☐` WEP `☑` WPA Personal `☐` WPA Enterprise | *Always use WPA Personal - nothing less*
| Security Key | Common passphrase | *Between 8 and 63 ASCII-encoded characters*
| Guest Policy | `☐` Apply guest policies (captive portal, guest authentication, access)
| **Advanced Options**
| Multicast and Broadcast Filtering | `☐` Block LAN to WLAN Multicast and Broadcast Data
| VLAN | `☑` Use VLAN `10` (2-4009)
| Fast Roaming (Beta) | `☑` Enable fast roaming
| Hide SSID | `☐` Prevent this SSID from being broadcast
| WPA Mode | `WPA2 Only` Encryption `AES/CCMP Only`
| Group Rekey Interval | `☑` Enable GTK rekeying every `3600` seconds
| User Group | Default
| UAPSD | `☐` Enable Unscheduled Automatic Power Save Delivery
| Scheduled | `☐` Enable WLAN schedule
| Multicast Enhancement | `☐` Enable multicast enhancement (IGMPv3)
| High Performance Devices | `☐` Connects high performance clients to 5 GHz only
| **802.11 Rate and Beacon Controls**
| DTIM Mode | `☑` Use default values
| DTIM 2G Period | Leave Default
| DTIM 5G Period | Leave Default
| 2G Data Rate Control | `☐` Enable minimum data rate control
| 5G Data Rate Control | `☐` Enable minimum data rate control
| **MAC Filter**
| Enabled | `☐` Enable filtering MAC addresses
| **RADIUS Mac Authentication**
| Enabled | `☐` Enable RADIUS MAC authentication
| RADIUS Profile | Leave Default
| MAC Address Format | Leave Default
| Empty Password | `☐` Allow empty password

And click `Save`.

Now repeat the procedure, using the above values except where shown (i.e Guest/IoT/NoT passphrase(s), Name/SSID, Guest Policy), creating new wireless networks on the following VLAN's:

| Create New Wireless Network | VLAN10 | VLAN20 | VLAN30 | VLAN40 | VLAN70 | VLAN110 | VLAN120 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Name/SSID | `name-open` | `name-smart` | `name-vpngate-world` | `name-vpngate-local` | `name-guest` | `name-iot` | `name-not` |
| Security Key | Common Passphrase | Common Passphrase |  Common Passphrase |  Common Passphrase |  **Guest Passphrase** |  **IoT Passphrase** |  **IoT Passphrase** |
| Guest Policy | `☐` | `☐` | `☐` | `☐` | `☑` | `☐` | `☐` |
| VLAN | `☑` Use VLAN `10` | `☑` Use VLAN `20` | `☑` Use VLAN `30` | `☑` Use VLAN `40` | `☑` Use VLAN `70` | `☑` Use VLAN `110` | `☑` Use VLAN `120` |

Your finished configuration must resemble the image below:

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_wireless.png)


## 2.0 UniFi Networks
We use VLANs to separate networks for easier management and to apply security policies.

### 2.1 Edit Default LAN network
First step is to edit your default LAN network configuration. Go to your UniFi controller `Settings` > `Networks` > `Actions - EDIT` and set the values as follows, remembering to click `Save` when done:

| Edit Network - LAN | Value | Notes
| :--- | :--- | :---
| Name | `LAN`
| Purpose | `Corporate`
| Network Group | `LAN`
| Port | LAN1
| Gateway/Subnet | `192.168.1.5/24` | *Remember to click `UPDATE DHCP RANGE`*
| Domain Name | localdomain
| IGMP Snooping | `☑` Enable IGMP Snooping
| DHCP Mode | `☑` DHCP Server
| DHCP Range | `192.168.1.150`-`192.168.1.250`
| DHCP Name Server | `☑`Manual `192.168.1.254` + `1.0.0.1`
| DHCP Lease Time | `86400`
| DHCP Gateway IP | `☑` Auto
| DHCP UniFi Controller | Leave Blank
| DHCP Guarding | `☐` Enable DHCP Guarding
| | `Trusted DCP server 1` | *Leave Blank*
| UPnP LAN | `☐` Enable UPnP LAN
| **Advanced DHCP Options**
| DHCP NTP Server | `☐` Enable DHCP NTP server
| NTP server 1 | 192.168.1.5
| NTP server 2 | Leave blank
| DHCP Network Boot | `☐` Enable network boot | *Only if you are running TFPBoot clients*
| DHCP Network Boot - IP | Leave blank | *i.e 192.168.1.10*
| DHCP Network Boot - Filename | Leave blank | *i.e /pxelinux.0*
| DHCP Time Offset | `☐` Enable DHCP time offset
| DHCP WPAD URL | Leave blank
| DHCP TFTP Server | Leave blank
| DHCP WINS Server | `☐` Enable DHCP WINS Server
| **Configure IPv6 Network**
| IPv6 Interface Type | `☐` None

And click `Save`.

### 2.2 Create UniFi Networks
Create the following new networks. Go to your UniFi controller `Settings` > `Networks` > `Create New Network` and set the values as follows, remembering to click `Save` at the end of each new network:

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_network_configuration.png)

On completion of the above task your your UniFi controller `Settings` > `Networks`  should resemble:

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_networks.png)

## 3.00 UniFi Routing & Firewall - Static Routes
Go to your UniFi controller `Routing & Firewall` and set the values as follows, remembering to click `Save` at the end.

### 3.01 Unifi Static Routes - Allow  access to VLAN30

| Create New Route | Value | Notes
| :--- | :--- | :---
| Name | `Allow  access to VLAN30`
| Enable | `☑` Enable this route
| Type| `☑` Static
| Network | `192.168.30.0/24`
| Distance | `1`
| Static Route Type | `☑` Next Hop
| Next Hop | `192.168.2.1`

And click `Save`

### 3.02 Unifi Static Routes - Allow  access to VLAN40

| Create New Route | Value | Notes
| :--- | :--- | :---
| Name | `Allow  access to VLAN40`
| Enable | `☑` Enable this route
| Type| `☑` Static
| Network | `192.168.40.0/24`
| Distance | `1`
| Static Route Type | `☑` Next Hop
| Next Hop | `192.168.2.1`

And click `Save`

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_routes_01.png)

## 4.00 UniFi Routing & Firewall  - Groups
You should create all your `Groups` before creating firewall rules. Go to your UniFi controller `Routing & Firewall` > `Firewall` > `Groups` and set the values as follows, remembering to click `Save` at the end.

### 4.01 Unifi Firewall Groups - All Chromecast Client IP addresses
Here you state your networks requiring Chromecast Access.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `All Chromecast addresses`
| Type | `☑` Address IPv4
| Address | `192.168.20.0/24` | *site-smart network - network with smart DNS*
|| `192.168.40.0/24` | *site-vpngate-local network*

And click `Save`

### 4.02 Unifi Firewall Groups - All IP Addresses

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `All IP addresses`
| Type | `☑` Address IPv4
| Address | `0.0.0.0/1`
|| `128.0.0.0/2`
|| `192.0.0.0/3`
|| `224.0.0.0/4`

And click `Save`

### 4.03 Unifi Firewall Groups - All local addresses

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `All local addresses`
| Type | `☑` Address IPv4
| Address | `192.168.1.0/24`
|| `192.168.10.0/24`
|| `192.168.20.0/24`
|| `192.168.30.0/24`
|| `192.168.40.0/24`
|| `192.168.50.0/24`
|| `192.168.60.0/24`
||| *192.168.70.0/24 is excluded because its a Guest network*
|| `192.168.80.0/24`
|| `192.168.90.0/24`
|| `192.168.110.0/24`
|| `192.168.120.0/24`

And click `Save`

### 4.04 Unifi Firewall Groups - All vpngate-local addresses

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `All vpngate-local addresses`
| Type | `☑` Address IPv4
| Address | `192.168.40.0/24`

And click `Save`

### 4.05 Unifi Firewall Groups - CCTV devices
Here you state all your CCTV camera IP addresses.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `CCTV devices`
| Type | `☑` Address IPv4
| Address | `192.168.120.0/24`

And click `Save`


### 4.06 Unifi Firewall Groups - CCTV ports
Here you state all your CCTV camera ports.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `CCTV ports`
| Type | `☑` Port
| Port | `tba`

And click `Save`

### 4.07 Unifi Firewall Groups - Chromecast devices
Here you state all your Chromecast device IP addresses including TV's.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `Chromecast devices`
| Type | `☑` Address IPv4
| Address | `192.168.20.151`

And click `Save`

### 4.08 Unifi Firewall Groups - Chromecast ports
Here you state all your Chromecast ports.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `Chromecast ports`
| Type | `☑` Port
| Port | `1900`
|| `5353`
|| `8008`
|| `8009`
|| `8443`

And click `Save`

### 4.09 Unifi Firewall Groups - Google DNS
Here you state Google DNS IP addresses. This is blocked on VLAN20.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `Google DNS`
| Type | `☑` Address IPv4
| Address | `8.8.8.8`
|| `8.8.4.4`

And click `Save`

### 4.10 Unifi Firewall Groups - Home Assistant Server

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `Home Assistant Server`
| Type | `☑` Address IPv4
| Address | `192.168.90.131`

And click `Save`

### 4.11 Unifi Firewall Groups - MQTT ports

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `MQTT ports`
| Type | `☑` Port
| Port | `1883`
|| `8883`

And click `Save`

### 4.12 Unifi Firewall Groups - NTP Ports

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `NTP ports`
| Type | `☑` Port
| Port | `123`

And click `Save`

### 4.13 Unifi Firewall Groups - Printing devices
Here you state your network printers IP address. You must assign a static IPv4 address either at the printer hardware or at the UniFi controller.

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `Printing devices`
| Type | `☑` Address IPv4
| Address | `192.168.1.91`

And click `Save`

### 4.14 Unifi Firewall Groups - Printing Ports
Here you state your printer models ports used for network printing. Best Google your printer model and add the ports specified. 

| Create New Group | Value | Notes
| :--- | :--- | :---
| Name | `Printing ports`
| Type | `☑` Port
| Port | `137` | *Brother Laser Printer port*
|| `631` | *Brother Laser Printer port*

And click `Save`

## 5.00 UniFi Routing & Firewall  - WAN IN
Nothing to do here.

## 6.00 UniFi Routing & Firewall  - WAN OUT
Nothing to do here.

## 7.00 UniFi Routing & Firewall  - WAN LOCAL
Nothing to do here.

## 8.00 UniFi Routing & Firewall  - LAN IN
Here we create most of our firewall rules under tha `LAN IN` Tab. Go to your UniFi controller `Routing & Firewall` > `Firewall` > `Rules IPv4` > `LAN IN` and set the values as follows, remembering to click `Save` at the end.

### 8.01 Unifi Firewall LAN IN - Allow All Established and Related Sessions

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Allow All Established and Related Sessions`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☑` Established
|| `☐` Invalid
|| `☑` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `All local addresses`
| Port Group | `Any`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `All local addresses`
| Port Group | `Any`

And click `Save`.

### 8.02 Unifi Firewall LAN IN - Accept all NTP Requests

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Accept all NTP Requests`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `All local addresses`
| Port Group | `Any`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Any`
| Port Group | `NTP Ports`

And click `Save`.

### 8.03 Unifi Firewall LAN IN - Allow NoT to MQTT

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Allow NoT to MQTT`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN-NoT`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Home Assistant Server`
| Port Group | `MQTT ports`

And click `Save`.

### 8.04 Unifi Firewall LAN IN - Allow IoT to Home Assistant

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Allow IoT to Home Assistant`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN-IoT`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Home Assistant Server`
| Port Group | `Any`

And click `Save`.

### 8.05 Unifi Firewall LAN IN - Allow Chromecast Broadcast

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Allow Chromecast Broadcast`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Chromecast devices`
| Port Group | `Any`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Any`
| Port Group | `Chromecast ports`

And click `Save`.

### 8.06 Unifi Firewall LAN IN - Block IoT from LAN

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Block IoT from LAN`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☑` Drop `☐` Reject `☐` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN-IoT`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN`
| Mac address | Leave blank

And click `Save`.

### 8.07 Unifi Firewall LAN IN - Block IoT from NoT

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Block IoT from NoT`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☑` Drop `☐` Reject `☐` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN-IoT`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN-NoT`
| Mac address | Leave blank

And click `Save`.

### 8.08 Unifi Firewall LAN IN - Block all NoT

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Block all NoT`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☑` Drop `☐` Reject `☐` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☐` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☐` Address/Port Group `☑` Network `☐` IP address
| Network | `LAN-NoT`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `All IP addresses`
| Port Group | `Any`

And click `Save`.

## 9.00 UniFi Routing & Firewall  - LAN OUT
Nothing to do here.

## 10.00 UniFi Routing & Firewall  - LAN LOCAL
Nothing to do here.

## 11.00 UniFi Routing & Firewall  - Guest IN
Here we create most of our firewall rules under tha `LAN IN` Tab. Go to your UniFi controller `Routing & Firewall` > `Firewall` > `Rules IPv4` > `GUEST IN` and set the values as follows, remembering to click `Save` at the end.

### 11.01 Unifi Firewall GUEST IN - Allow Chromecast devices In

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Allow Chromecast devices In`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☐` New
|| `☑` Established
|| `☐` Invalid
|| `☑` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Chromecast devices`
| Port Group | `Any`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `All Chromecast Client IP addresses`
| Port Group | `Chromecast ports`

And click `Save`.

### 11.02 Unifi Firewall GUEST IN - Allow Printer devices In

| Create New Rule | Value | Notes
| :--- | :--- | :---
| Name | `Allow Printer devices In`
| Enabled | `☑` On
| Rule Applied | `☑` Before predefined rules `☐` After predefined rules
| Action | `☐` Drop `☐` Reject `☑` Accept
| IPv4 Protocol | `☑` All
|| `☐` TCP
|| `☐` UDP
|| `☐` TCP and UDP
|| `☐` ICMP
|| `☐` Choose a protocol by name
|| `☐` Enter a protocol number
| **Advanced**
| Logging | `☐` Enable logging
| States | `☑` New
|| `☑` Established
|| `☐` Invalid
|| `☐` Related
| IPsec | `☑` Don't match on IPsec packets
|| `☐` Match inbound IPsec packets
|| `☐` Match inbound non-IPsec packets
| **Source**
| Source Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Printing Devices`
| Port Group | `Any`
| Mac address | Leave blank
| **Destination**
| Destination Type | `☑` Address/Port Group `☐` Network `☐` IP address
| IPv4 Address Group | `Any`
| Port Group | `Printing Ports`

And click `Save`.

## 12.00 UniFi Routing & Firewall  - GUEST OUT
Nothing to do here.

## 13.00 UniFi Routing & Firewall  - GUEST LOCAL
Nothing to do here.

## 14.00 UniFi Routing & Firewall  - Port Forwarding
Here we create our port forwarding rules. Go to your UniFi controller `Routing & Firewall` > `Port Forwarding` and set the values as follows, remembering to click `Save` at the end.

### 14.01 Unifi Port Forwarding - HAProxy

| Create New Port Forward Rule | Value | Notes
| :--- | :--- | :---
| Name | `HAProxy`
| Enabled | `☑` Enable this port forward rule
| From | `☑` Anywhere `☐` Limited
| Port | `80`
| Forward IP | `192.168.2.1`
| Forward Port | `80`
| Protocol | `☐` Both `☑` TCP `☐` UDP
| Logs | `☐` Enable logging

And click `Save`.

### 14.02 Unifi Port Forwarding - HAProxy

| Create New Port Forward Rule | Value | Notes
| :--- | :--- | :---
| Name | `HAProxy`
| Enabled | `☑` Enable this port forward rule
| From | `☑` Anywhere `☐` Limited
| Port | `443`
| Forward IP | `192.168.2.1`
| Forward Port | `443`
| Protocol | `☐` Both `☑` TCP `☐` UDP
| Logs | `☐` Enable logging

And click `Save`.

### 14.03 Unifi Port Forwarding - Syncthing - listening

| Create New Port Forward Rule | Value | Notes
| :--- | :--- | :---
| Name | `Syncthing - listening`
| Enabled | `☑` Enable this port forward rule
| From | `☑` Anywhere `☐` Limited
| Port | `22000`
| Forward IP | `192.168.80.122`
| Forward Port | `22000`
| Protocol | `☐` Both `☑` TCP `☐` UDP
| Logs | `☐` Enable logging

And click `Save`.

## 15.00 UniFi IPS
Coming soon.

## 16.00 UniFi DPI
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_dpi.png)

## 17.00 UniFi Guest Control

| Guest Policies | Value | Notes
| :--- | :--- | :---
| Guest Portal | `☐` Enable Guest Portal
| **Access Control**
| Pre-Authorization Access | 
| Post-Authorization Restrictions | `192.168.0.0/16`
|| `172.16.0.0/12`
|| `10.0.0.0/8`
|| `192.168.1.206/32`

## 18.00 UniFi Profiles
Coming soon.

## 19.00 UniFi Services
Go to your UniFi controller `Settings` > `Services` and set the values as follows, remembering to click `Save` at the end.

### 19.01 UniFi Services - DHCP

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_dhcp.png)

### 19.02 UniFi Services - MDNS
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_mdns.png)

### 19.03 UniFi Services - UPNP
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_upnp.png)

### 19.04 UniFi Services - NTP
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_ntp.png)

### 19.05 UniFi Services - Scheduled Upgrades
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_upgrades.png)

## 20.00 UniFi Admins
Coming soon.

## 21.00 UniFi User Groups
Coming soon.

## 22.00 UniFi Controller
Coming soon.

## 23.00 UniFi User Interface
Coming soon.

## 24.00 UniFi Notifications
Coming soon.

## 25.00 UniFi Cloud Access
Coming soon.

## 26.00 UniFi Elite Device
Coming soon.

## 27.00 UniFi Maintenance
Coming soon.

## 28.00 UniFi Auto Backup
Go to your UniFi controller `Settings` > `Auto Maintenance` and set the values as follows, remembering to click `Save` at the end.
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_autobackup.png)

---

## 00.00 Patches & Fixes
Coming soon.

