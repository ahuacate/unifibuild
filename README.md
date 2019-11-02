# UniFi Build
This recipe is my UniFi network configuration. These instructions are focused on setting up your `Networks`, `Wireless Networks` and `Routing & Firewall` so they work.

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
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or goes down)
- [x] Network DHCP server is `192.168.1.5`
- [x] A DDNS service is fully configured and enabled (I recommend you use the free Synology DDNS service)
- [x] A ExpressVPN account (or any preferred VPN provider) is valid and its smart DNS feature is working (public IP registration is working with your DDNS provider)

Other Prerequisites are:
- [x] Synology NAS is `192.168.1.10`
- [x] Synology NAS is installed with Synology Virtual Machine Manager
- [x] Synology NAS is configured, including NFS, as per [synobuild](https://github.com/ahuacate/synobuild)

Tasks to be performed are:
- [ ] 1.0 Proxmox Base OS Installation

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
| | Password `Random character password ONLY i.e cA(8&KxjLHz8s4?A`
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

Now repeat the procedure, using the above values except where shown (i.e Guest passphrase, Name/SSID, Guest Policy), creating new wireless networks on the following VLAN's:

| Create New Wireless Network | VLAN10 | VLAN20 | VLAN30 | VLAN40 | VLAN70 | VLAN110 | VLAN120 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Name/SSID | `name-open` | `name-smart` | `name-vpngate-world` | `name-vpngate-local` | `name-guest` | `name-iot` | `name-not` |
| Security Key | Common Passphrase | Common Passphrase |  Common Passphrase |  Common Passphrase |  **Guest Passphrase** |  IoT Passphrase |  IoT Passphrase |
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

On completion of the above task your UniFi `Netowrks` WebGUI should resemble:

![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_networks.png)

## 3.00 UniFi Routing & Firewall
Coming soon.

## 4.00 UniFi IPS
Coming soon.

## 5.00 UniFi DPI
Coming soon.

## 6.00 UniFi Guest Control
Coming soon.

## 7.00 UniFi Profiles
Coming soon.

## 8.00 UniFi Services
Go to your UniFi controller `Settings` > `Services` and set the values as follows, remembering to click `Save` at the end.

### 8.01 UniFi Services - DHCP
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_dhcp.png)

### 8.02 UniFi Services - MDNS
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_mdns.png)

### 8.03 UniFi Services - UPNP
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_upnp.png)

### 8.03 UniFi Services - NTP
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_ntp.png)

### 8.03 UniFi Services - Scheduled Upgrades
![alt text](https://raw.githubusercontent.com/ahuacate/unifibuild/master/images/unifi_services_upgrades.png)
