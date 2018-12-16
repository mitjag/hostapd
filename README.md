# hotstapd
Setting hostapd in Armbian using USB WiFi adapter

# Hardware
OrangePi Prime

http://www.orangepi.org/OrangePiPrime/

Asus USB-N14 Wireless-N300 USB Adapter

https://www.asus.com/Networking/USBN14/


# Armbian image

Let start with creating latest and fresh Armbian image

https://docs.armbian.com/Developer-Guide_Build-Preparation/

Install Ubutnu 18.04 LTS
and follow armbian build instructions.


Full OS image
Target bionic OS release


Etcher to burn card
https://www.balena.io/etcher/

Login to armbian with default user: root/1234

# Armbian setup

After pluging in USB WiFi adapter check dmesg:

```
root@skywirenode3:~# dmesg
...
[   49.861412] usb 5-1: new high-speed USB device number 2 using ehci-platform
[   50.033958] usb 5-1: New USB device found, idVendor=0b05, idProduct=17e8
[   50.033974] usb 5-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[   50.033985] usb 5-1: Product: 802.11 n WLAN
[   50.033995] usb 5-1: Manufacturer: Ralink
[   50.034006] usb 5-1: SerialNumber: 1.0
[   50.256538] usb 5-1: reset high-speed USB device number 2 using ehci-platform
[   50.422953] ieee80211 phy1: rt2x00_set_rt: Info - RT chipset 5392, rev 0223 detected
[   50.461296] ieee80211 phy1: rt2x00_set_rf: Info - RF chipset 5372 detected
[   50.471822] ieee80211 phy1: Selected rate control algorithm 'minstrel_ht'
[   50.472831] usbcore: registered new interface driver rt2800usb
[   50.489362] rt2800usb 5-1:1.0 wlx4cedfbd746e2: renamed from wlan1
[   50.555693] IPv6: ADDRCONF(NETDEV_UP): wlx4cedfbd746e2: link is not ready
[   50.555888] ieee80211 phy1: rt2x00lib_request_firmware: Info - Loading firmware file 'rt2870.bin'
[   50.557688] ieee80211 phy1: rt2x00lib_request_firmware: Info - Firmware detected - version: 0.36
[   50.833442] IPv6: ADDRCONF(NETDEV_UP): wlx4cedfbd746e2: link is not ready
[   50.895897] IPv6: ADDRCONF(NETDEV_UP): wlx4cedfbd746e2: link is not ready
root@skywirenode3:~#
```

Validate that new adapter is present use ip or ifconfig tool

```
root@skywirenode3:~# ip a l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 02:01:4b:79:5b:49 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.220/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::1:4bff:fe79:5b49/64 scope link
       valid_lft forever preferred_lft forever
3: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 10:a4:be:21:aa:68 brd ff:ff:ff:ff:ff:ff
4: wlx4cedfbd746e2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 4c:ed:fb:d7:46:e2 brd ff:ff:ff:ff:ff:ff
```

```
root@skywirenode3:~# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:01:4b:79:5b:49
          inet addr:192.168.1.220  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::1:4bff:fe79:5b49/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:550 errors:0 dropped:0 overruns:0 frame:0
          TX packets:283 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:79058 (79.0 KB)  TX bytes:85754 (85.7 KB)
          Interrupt:25

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:100 errors:0 dropped:0 overruns:0 frame:0
          TX packets:100 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:8192 (8.1 KB)  TX bytes:8192 (8.1 KB)

wlan0     Link encap:Ethernet  HWaddr 10:a4:be:21:aa:68
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

wlx4cedfbd746e2 Link encap:Ethernet  HWaddr 4c:ed:fb:d7:46:e2
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

Check if your adapter supports AP mode with command 'iw list' under section 'Supported interface modes' there should be 'AP' mode listed:

```
ctrl_interface_group=0root@skywirenode3:/etc# iw list
Wiphy phy1
        max # scan SSIDs: 4
        max scan IEs length: 2257 bytes
        Retry short long limit: 2
        Coverage class: 0 (up to 0m)
        Device supports RSN-IBSS.
        Supported Ciphers:
                * WEP40 (00-0f-ac:1)
                * WEP104 (00-0f-ac:5)
                * TKIP (00-0f-ac:2)
                * CCMP (00-0f-ac:4)
                * 00-0f-ac:10
                * GCMP (00-0f-ac:8)
                * 00-0f-ac:9
        Available Antennas: TX 0 RX 0
        Supported interface modes:
                 * IBSS
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor
                 * mesh point
        Band 1:
...
```


You can see in hostapd.conf default configuration for wlan0 change it to new adapter as listed with ip or ifconfig (eg.: my usb wifi adapter is: wlx4cedfbd746e2)

```
root@skywirenode3:/etc# cat /etc/hostapd.conf
#
# armbian hostapd configuration example
#
# nl80211 mode
#

ssid=ARMBIAN
interface=wlx4cedfbd746e2
hw_mode=g
channel=5
bridge=br0
driver=nl80211

logger_syslog=0
logger_syslog_level=0
wmm_enabled=1
wpa=2
preamble=1

wpa_psk=66eb31d2b48d19ba216f2e50c6831ee11be98e2fa3a8075e30b866f4a5ccda27
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
auth_algs=1
macaddr_acl=0

noscan=1

### IEEE 802.11n
#ieee80211n=1
#ht_capab=
#country_code=US
#ieee80211d=1
### IEEE 802.11n

### IEEE 802.11a
#hw_mode=a
### IEEE 802.11a

### IEEE 802.11ac
#ieee80211ac=1
#vht_capab=[MAX-MPDU-11454][SHORT-GI-80][TX-STBC-2BY1][RX-STBC-1][MAX-A-MPDU-LEN-EXP3]
#vht_oper_chwidth=1
#vht_oper_centr_freq_seg0_idx=42
### IEEE 802.11ac

# controlling enabled
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0root@skywirenode3:/etc#
```


To enable service to load hostapd.conf uncomment line with DAEMON_CONF parameter.

```
root@skywirenode3:/etc# cat /etc/default/hostapd
# Defaults for hostapd initscript
#
# See /usr/share/doc/hostapd/README.Debian for information about alternative
# methods of managing hostapd.
#
# Uncomment and set DAEMON_CONF to the absolute path of a hostapd configuration
# file and hostapd will be started during system boot. An example configuration
# file can be found at /usr/share/doc/hostapd/examples/hostapd.conf.gz
#
DAEMON_CONF="/etc/hostapd.conf"

# Additional daemon options to be appended to hostapd command:-
#       -d   show more debug messages (-dd for even more)
#       -K   include key data in debug messages
#       -t   include timestamps in some debug messages
#
# Note that -B (daemon mode) and -P (pidfile) options are automatically
# configured by the init.d script and must not be added to DAEMON_OPTS.
#
#DAEMON_OPTS=""
root@skywirenode3:/etc#
```

In folder /etc/network there are network example configuration files copy interfaces.hostapd over interfaces and update parameters
```
root@skywirenode3:/etc/network# ls /etc/network
if-down.d       if-pre-up.d  interfaces          interfaces.d        interfaces.espressobin  interfaces.network-manager  interfaces.r1        interfaces.r1switch
if-post-down.d  if-up.d      interfaces.bonding  interfaces.default  interfaces.hostapd      interfaces.org              interfaces.r1router
root@skywirenode3:/etc/network#
root@skywirenode3:/etc/network# cp interfaces.hostapd interfaces
root@skywirenode3:/etc/network# cat interfaces
auto lo br0
iface lo inet loopback

auto eth0
iface eth0 inet manual

auto wlan0
iface wlan0 inet manual

auto wlx4cedfbd746e2
iface wlx4cedfbd746e2 inet manual

iface br0 inet dhcp
#bridge_ports eth0 wlan0
bridge_ports eth0 wlx4cedfbd746e2
#hwaddress ether # will be added at first boot
root@skywirenode3:/etc/network#
```

Reboot and enjoy custom Access Point with Armbian HotSpot using Wifi USB Adapter.
