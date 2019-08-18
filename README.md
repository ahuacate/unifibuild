# UniFi Build
This recipe is my UniFi network configuration. These instructions are focused on setting up your `Networks`, `Wireless Networks` and `Routing & Firewall` so they work. Basic or general UniFi Controller WebGUI configuration is not covered.

The hardware in this recipe uses:
>  *  1x Ubiquiti UniFi Security Gateway 3P (USG)
>  *  1x Ubiquiti UniFi Switch 24
>  *  1x Ubiquiti UniFi Switch 8 POE-150W
>  *  1x Ubiquiti UniFi AP-AC-Pro
>  *  1x Synology DS1515+ with 4x NICs; and,
>  *  Various hardware for a Proxmox cluster (Qotom Mini PC Q500G6-S05).

Obviously you can modify these instructions to meet your own hardware requirements.

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

## 1.0 UniFi Controller
To be completed.

## 2.0 UniFi Networks
I have used VLANs to separate my network for easier management and to apply security policies.

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
| DHCP UniFi Controller | Leave Default
| **DHCP Guarding** | `☐` Enable DHCP Guarding
| Trusted DCP server 1 | Leave Blank
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

### 2.2 Create UniFi VLAN Networks

| Create New Network | VLAN2 | VLAN20 | VLAN30 | VLAN40 | VLAN50 | VLAN60 | VLAN70 | VLAN80 | VLAN90 | VLAN100 | VLAN110 | VLAN120
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---:
| Name | `VPN-egress` | `LAN-smart` | `LAN-vpngate-world` | `LAN-vpngate-local` | `LAN-media` | `LAN-vpnserver` | `LAN-guest` | `LAN-homelab` | `LAN-privatelab` | `LAN-hass` | `LAN-iot` | `LAN-security` 
| Purpose |`Guest` |`Corporate`|`VLAN Only`|`VLAN Only`|`Corporate`|`Corporate`|`Guest`|`Corporate`|`Corporate`|`Corporate`|`Corporate`|`Corporate
| Network Group |`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|`LAN`|
| Port |LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1|LAN1
| VLAN | 2 |20|30|40|50|60|70|80|90|100|100|110|120
| Gateway/Subnet |`192.168.2.5/28`| 
| Domain Name | Leave blank
| IGMP Snooping |`☑` Enable IGMP Snooping
| DHCP Mode |`☑` DHCP Server
| DHCP Range |`192.168.2.1`-`192.168.2.14`
| DHCP Name Server |`☑` Auto
| DHCP Lease Time |`86400`
| DHCP Gateway IP |`☑` Auto
| DHCP UniFi Controller |Leave Default
| **DHCP Guarding** |`☐` Enable DHCP Guarding
| Trusted DCP server 1 |Leave Blank
| UPnP LAN |`☐` Enable UPnP LAN
| **Advanced DHCP Options**
| DHCP NTP Server |`☐` Enable DHCP NTP server
| NTP server 1 |192.168.1.5
| NTP server 2 |Leave blank
| DHCP Network Boot |`☐` Enable network boot |
| DHCP Network Boot - IP |Leave blank |
| DHCP Network Boot - Filename |Leave blank |
| DHCP Time Offset |`☐` Enable DHCP time offset
| DHCP WPAD URL |Leave blank
| DHCP TFTP Server |Leave blank
| DHCP WINS Server |`☐` Enable DHCP WINS Server
| **Configure IPv6 Network**
| IPv6 Interface Type |`☐` None
