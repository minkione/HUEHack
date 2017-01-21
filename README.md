# HUE HACK: Soft Link Button

# Description
I go really tired of walking a flight of stairs to push a button on the bridge for the x time. So I though: _if only I could use some web call to do that, hmmm_ Luckily we can!!!

_Thanks to all the people in the references!!!_

## Other nice stuff!
 * [iBrew](https://github.com/Tristan79/iBrew) iKettle, iKettle 2.0 and Smarter Coffee Interface
 * [iSamsungTV](https://github.com/Tristan79/iSamsungTV) the command line interface to Samsung TV series C, D, E, F and Blue Ray Disc Players with Smart Hub feature.
 * [Medisana Scale](https://github.com/keptenkurk/BS440) [Domoticz](http://domoticz.com/) bridge to Medisana BS440, BS430,... weight scales.
 * [Xiaomi Mi Plant Sensor](https://github.com/Tristan79/miflora) with [Domoticz](http://domoticz.com/) bridge
 * [Vento](https://github.com/Tristan79/Vento)  The itho, duco, orcon, zehnder, storkair: arduino [mysensors 2.0](https://www.mysensors.org) controller!!!


## Rooting!

First pin2pwn your way into the bridge... see all the references for more detail information. This is a fast walkthrough! Read the references first!!! 

Let's go! Open it up!

![screw](https://raw.githubusercontent.com/Tristan79/HUEHACK/master/screw.png)

Solder a header to it (or not, but makes life easier :-)

![pins](https://raw.githubusercontent.com/Tristan79/HUEHACK/master/pins.jpg)

Need on of these (1.50 euro :-)

![usbserial](https://raw.githubusercontent.com/Tristan79/HUEHACK/master/usbserial.jpg)

Make sure it's jumper selects 3.3v and connect it with some dupont wires, 

```
USBSERIAL  <--> BRIDGE
GRD        <--> 1
TX         <--> 4 (RX)
RX         <--> 5 (TX)
```

Use a serial monitor/terminal app (I used the one build in Arduino) to look if you see any chatter. If so short it before it loads the flash... (look for the word bee ;-)

![short](https://raw.githubusercontent.com/Tristan79/HUEHACK/master/short.jpg)


With a prompt. Make your own password on ubuntu (or some other distro that has that command) with the mkpasswd command. Pick MD5 if you want to login to the bridge with ssh. SHA-256 won't work.

```
mkpasswd --method=MD5
```

Example output password: toor
```
$1$trHrRIqN$2iZEbgsyCy12f6tNeKvO41)
```

if not using ssh you can use SHA-256

```
mkpasswd --method=SHA-256
```

Example output password: toor
```
$5$5ijpVxNPFB$2Viqp.XSTeIHOaV.AKs8Fx.oqytyr5qySFIbr5iza59
```

Whatever you choose set it up with (your own password):

```
setenv security '$1$trHrRIqN$2iZEbgsyCy12f6tNeKvO41'
setenv bootdelay 3
saveenv
```

Reboot... when finished. Login to the terminal 

### SSH


Only works with MD5 passwords...

```
cp /etc/rc.local /etc/rc.local.orig
echo -e "iptables -I INPUT -p tcp --dport 22 --syn -j ACCEPT" > /etc/rc.local
cp /etc/config/dropbear /etc/config/dropbear.orig
echo -e "\toption Rootl.Login '1'" >> /etc/config/dropbear
```

### Telnet

Only use if you do not use SSH!

```
echo -e "iptables -I INPUT -p tcp --dport 23 --syn -j ACCEPT" > /etc/rc.local
echo -e "telnetd" >> /etc/rc.local
```

Reboot... 

After this you can use either telnet or ssh into it... pfff ;-)

## Setup!

### Package Mananger

The alternative has less nice stuff :-)

```
cp /etc/opkg.conf /etc/opkg.conf.orig
# Alternative: echo -e "src/gz bsb002 http://downloads.openwrt.org/backfire/10.03.1/ar71xx/packages" > /etc/opkg.conf
echo -e "src/gz bsb002 http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/packages/" > /etc/opkg.conf
echo -e "dest root /" >> /etc/opkg.conf
echo -e "dest ram /tmp" >> /etc/opkg.conf
echo -e "lists_dir ext /var/opkg-lists" >> /etc/opkg.conf
echo -e "option overlay_root /overlay" >> /etc/opkg.conf
opkg update
```

### Nano!

```
opkg install nano
echo -e "export TERM=xterm" >> /etc/profile
```

### Wifi (optional)

```
cp /etc/config/wireless /etc/config/wireless.orig
opkg update
opkg install wpa-supplicant
nano /etc/config/wireless
```

Make it like this:

```
config wifi-device  radio0
	option type     mac80211
	option channel  11
	option hwmode	11ng
	option path	'platform/qca953x_wmac'
	list ht_capab	LDPC
	list ht_capab	SHORT-GI-20
	list ht_capab	SHORT-GI-40
	list ht_capab	TX-STBC
	list ht_capab	RX-STBC1
	list ht_capab	DSSS_CCK-40
	option htmode	HT20
    
config wifi-iface
        option device   radio0
        option network  wlan
        option mode     sta
        option ssid     YourSSID
        option encryption psk2
        option key      YourPassword
```

Save & commit!

```
uci commit wireless
wifi
nano /etc/config/network
```

Add 

```
config interface 'wlan'
	option ifname 'wlan0'
        option ipaddr '192.168.1.2'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option proto 'dhcp'
        option hostname 'Philips-hue'
```

#### Firewall (optional)

```
nano /etc/config/firewall
```

Open it all up add
```
config zone ‘wlan’
    option name ‘wlan’
    list network ‘wlan’
    option input ‘ACCEPT’
```

Or copy all entries :-)

```
config zone 'wlan'
        option name             wlan
        list   network          'wlan'
        option input            REJECT
        option output           ACCEPT
        option forward          REJECT
        option masq             1
        option mtu_fix          1

config rule 'clip_wifi'
        option name             Allow-CLIP-wifi
        option src              wlan
        option proto            tcp
        option dest_port        80
        option target           ACCEPT
        option family           ipv4

config rule 'factory_wifi'
	option name		Allow-Factory-wifi
	option src		wlan
	option proto		tcp
	option dest_port	30000
	option target		ACCEPT
	option family		ipv4

config rule 'ssdp_wifi'
	option name		Allow-SSDP-wifi
	option src		wlan
	option proto		udp
	option dest_port	1900
	option target		ACCEPT
	option family		ipv4
 
config rule 'mdns_ipv4_wifi'
        option name             Allow-mdns-ipv4-wifi
        option src              wlan
        option proto            udp
        option dest_port        5353
        option target           ACCEPT
        option family           ipv4

config rule 'mdns_ipv6_wifi'
        option name             Allow-mdns-ipv6-wifi
        option src              wlan
        option proto            udp
        option dest_port        5353
        option target           ACCEPT
        option family           ipv6
 
config rule 'hap_ipv4_wifi'
	option name		Allow-hap-ipv4-wifi
	option src		wlan
	option proto		tcp
	option dest_port	8080
	option target		ACCEPT
	option family		ipv4
 
config rule 'hap_ipv6_wifi'
	option name		Allow-hap-ipv6-wifi
	option src		wlan
	option proto		tcp
	option dest_port	8080
	option target		ACCEPT
	option family		ipv6

config rule 'dhcp_renew_wifi'
        option name             Allow-DHCP-Renew-wifi
        option src              wlan
        option proto            udp
        option dest_port        68
        option target           ACCEPT
        option family           ipv4

config rule 'ping_wifi'
        option name             Allow-Ping-wifi
        option src              wlan
        option proto            icmp
        option icmp_type        echo-request
        option family           ipv4
        option target           ACCEPT

config rule 'dhcpv6_wifi'
	option name		Allow-DHCPv6-wifi
	option src		wlan
	option proto		udp
	option src_ip		fe80::/10
	option src_port		547
	option dest_ip		fe80::/10
	option dest_port	546
	option family		ipv6
	option target		ACCEPT

config rule 'icmpv6_input_wifi'
	option name		Allow-ICMPv6-Input-wifi
	option src		wlan
	option proto	icmp
	list icmp_type		echo-request
	list icmp_type		echo-reply
	list icmp_type		destination-unreachable
	list icmp_type		packet-too-big
	list icmp_type		time-exceeded
	list icmp_type		bad-header
	list icmp_type		unknown-header-type
	list icmp_type		router-solicitation
	list icmp_type		neighbour-solicitation
	list icmp_type		router-advertisement
	list icmp_type		neighbour-advertisement
	option limit		1000/sec
	option family		ipv6
	option target		ACCEPT

config rule 'icmpv6_forward_wifi'
        option name             Allow-ICMPv6-Forward-wifi
        option src              wlan
        option dest             *
        option proto            icmp
        list icmp_type          echo-request
        list icmp_type          echo-reply
        list icmp_type          destination-unreachable
        list icmp_type          packet-too-big
        list icmp_type          time-exceeded
        list icmp_type          bad-header
        list icmp_type          unknown-header-type
        option limit            1000/sec
        option family           ipv6
        option target           ACCEPT
```

### Python/Lua

```
opkg update
opkg install lua
opkg install python
```

## Putting it al together...

### Some extra: Toggling leds :-)

#### World Network LED
```
echo 1 > /sys/class/leds/bsb002\:blue\:portal/brightness
delay 1
echo 0 > /sys/class/leds/bsb002\:blue\:portal/brightness
```

#### Network LED

```
echo 1 >/sys/class/leds/bsb002\:blue\:network/brightness
delay 1
echo 0 >/sys/class/leds/bsb002\:blue\:network/brightness
```


#### Always accept

```
echo "[btn_link,pressed]" > /var/hue-ipbridge/button_in
```

Just never release it :-)
Possible to add it to: /etc/rc.local


### Simulate bridge press

```
echo "[btn_link,pressed]" > /var/hue-ipbridge/button_in
echo "[btn_link,released]" > /var/hue-ipbridge/button_in
```

### The Server


```
nano /etc/ButtonLink.py
```

Add the following code

```
__version__ = "0.1"
__all__ = ["ButtonLink"]

import os
import BaseHTTPServer
import urllib
import shutil

try:
    from cStringIO import StringIO
except ImportError:
    from StringIO import StringIO

class SimpleHTTPRequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):

    server_version = "ButtonLinkHTTP/" + __version__

    def do_GET(self):
        f = StringIO()
        f.write("<html>\n<title>HUE Soft Link Button</title>\n")
        f.write("<body>Toggled!</body>\n</html>\n")
        os.system("echo \"[btn_link,pressed]\" > /var/hue-ipbridge/button_in")
        os.system("echo \"[btn_link,released]\" > /var/hue-ipbridge/button_in")
        length = f.tell()
        f.seek(0)
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.send_header("Content-Length", str(length))
        self.end_headers()
        shutil.copyfileobj(f, self.wfile)
        if f:
            f.close()


def test(HandlerClass = SimpleHTTPRequestHandler,
         ServerClass = BaseHTTPServer.HTTPServer):
    BaseHTTPServer.test(HandlerClass, ServerClass)

if __name__ == '__main__':
    test()

```

### Firewall rule

```
echo -e "iptables -I INPUT -p tcp --dport 8000 --syn -j ACCEPT" >> /etc/rc.local
```

### Persistence

```
echo -e "python /etc/ButtonLink.py" >> /etc/rc.local
```

### Usage

Surf to 

```
http://IP_HUE_BRIGE:8000/
```

to toggle the button...

# Final Though

You're HUE bridges is rooted so you can probably do more fun stuff with it. Note that a factory reset will wipe any trace of any mod you made. It will survive an update (so far)

Well thats-sa-bout-it...

Signed TRiXWooD

# References

* [Rxsegers](https://medium.com/@rxseger/enabling-the-hidden-wi-fi-radio-on-the-philips-hue-bridge-2-0-42949f0154e1#.40l92j7uk)
* [ColinoFlynn](http://colinoflynn.com/2016/07/getting-root-on-philips-hue-bridge-2-0/)
* [OpenWRT](https://forum.openwrt.org/viewtopic.php?id=66346)
* [Reddit](https://www.reddit.com/r/Hue/comments/3x12y6/jailbreaking_the_v2_hub/)
* [Reddit](https://www.reddit.com/r/Hue/comments/4xyaak/experiments_with_enabling_the_builtin_wifi_radio/)
* [Tarlab Oulu](https://www.jkry.org/ouluhack/HackingHueBridge#Getting_root_shell_.28HW_hacking.29)
