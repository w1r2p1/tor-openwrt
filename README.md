# Implementation of WiFi hotspot and middle relay with anonymity network #

### Using Tor network on TP-LINK WDR4300 with OpenWRT ###
Run Chen, (run.chen@mail.polimi.it) <br/>
Jorge Díez de la Fuente, (jorge.diez.delafuente@alumnos.upm.es)

Report for the master course of Embedded Systems <br/>
Reviser: PhD. Federico Terraneo (federico.terraneo@polimi.it)

Received: July, 2014

## Abstract ##

Increasing network users and the continuing growth of information technology have made people to pay attention in security issues. Our project focuses on providing more privacy and protection to small and medium WiFi hotspot provider while they’re providing internet and to users as well they’re surfing internet. <br/>
As you know, WiFi hotspot provide us a free internet connection in specific areas around the world, but in most cases is lack of sufficient privacy for users. We implemented a WiFi hotspot with tor network, and provide users a higher level of privacy and anonymity. On the other hand, this implementation protects the hotspot provider from any illegality which is committed in its network. <br/>
Finally we will present our conclusion based on test results, we will also discuss our opinion about this implementation, pros and cons. We also comment some security issues about this implementation, we describe those problem and propose a solution.

## 1. Introducction ##

The purpose of this project is implementing a middle node and wifi hotspot through tor network in a commercial router and evaluate its performance. First we will make a brief introduction of each part of the project. We start explaining what is tor, different types of tor relay and why we want to implement a wifi hotspot with tor.

### 1.1. Tor network ###

> Tor (acronym for The Onion Router) is free software for enabling online anonymity and resisting censorship. Tor directs Internet traffic through a free, worldwide, volunteer network consisting of more than five thousand relays to conceal a user's location or usage from anyone conducting network surveillance or traffic analysis. Using Tor makes it more difficult for Internet activity to be traced back to the user: this includes "visits to Web sites, online posts, instant messages, and other communication forms". Tor's use is intended to protect the personal privacy of users, as well as their freedom and ability to conduct confidential communication by keeping their Internet activities from being monitored.

Wikipedia.  [TOR anonymity network](http://en.wikipedia.org/wiki/Tor_(anonymity_network))

Tor network is consist of Entry node, Middle node and Exit node. We are focussing on Middle node in our project.

![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/tornet.png "Tor Network")

### 1.2 Middle relays ###

> Middle relays which receive traffic and pass it to another relay. Middle relays increase robustness and accelerate speed of the Tor network without making the owner of the relay look like the source of the traffic. Middle relays advertise their presence to the rest of the Tor network, so that any Tor user can connect to them. Even if a malicious user employs the Tor network to do something illegal, the IP address of a middle relay will not show up as the source of the traffic. That means a middle relay is generally safe to run in your home, in conjunction with other services, or on a computer with your personal files.

TOR project. [TOR Relay Debian](https: //www.torproject.org/docs/tor-relay-debian.html.en/)

### 1.3 WIFI hotspot through TOR ###

We want to share our wifi access point. When the traffic is routing through tor network, we can obtain a manner to provide security to the WIFI hotspot-owner since the traffic of people who connected to our system is spreading through Tor and avoiding by someone misuses our network. <br/>
The main advantages are: <br/>
Traffic is routed encryted through the tor-network and when the packet reaches "outside" there is no way to know from where come from. <br/>
Users connected to hotspot don't know anything about router network structure(ip, lan, .., etc. it provides more privacy for owner.

## 2. Implementation ##

We wanted to bring Internet access to our neighbours. The best solution was WiFi so we planed to build an open wireless network. <br/>
At this point we found two security issues: The first one was that our home network servers weren't secure at all behind the NAT, so we had to build a second network isolated from the first one. <br/>
The second issue was that we were sharing our IP with unknown people and we didn't trust their purpose so we decided to anonymize the traffic generated by the public hotspot. We employed Tor software because it's a mature open source project and we were acquainted with it. All the traffic comming from the hotspot will be passed through a transparent proxy to Tor entry relay, cross the tor network ant get the net. With this setup here is no need of configuration on the client/user side. <br/>
We wanted also to support the Tor project so we decided to run also a middle relay on the router. <br/>
We think that we weren't alone with this problem so we tried to keep simple and our solution affordable to non professional users, that's why we decided to employ a single device.

We also want to implement a middle relay in the same router. The goal of this project is to know if is possible to implement both system together.


### 2.1 Materials ###

We employed a TP-Link WDR4300 (Atheros 9344 MIPS, 128MB RAM) plus a usb stick 1GB storage. The internal storage wasn't large enough for Tor so we plugged an usb stick, this tiny piece of hardware doesn't increase the volume and the price of our platform. <br/>
On the software side we employed OpenWRT Attitude Adjustment and tor 0.2.4.22-1 <br/>
In one word, we kept simple.

### 2.2 Development ###

Before we start, we need a commercial router with OpenWRT and a usb stick with a two partitions: the first ext4 and the second swap format. Install the following packages on the router: tor, iptables-mod-nat and iptables-mod-nat-extra, then plug a usb stick.

TOR project. [TOR router set up notes](https://trac.torproject.org/projects/tor/wiki/doc/Torouter/OpenWRT_setup_notes) <br/>
zBlackhatworld. [Speeding up tor](http://www.blackhatworld.com/blackhat-seo/proxies/3349-speeding-up-tor.html)

Let's configure the Hotspot:

Build a new network,

/etc/config/network <br/>
```
config 'interface' 'tor'
	option proto 'static'
	option ipaddr '192.168.2.1'
	option netmask '255.255.255.0'
```

Configure the DHCP:

/etc/config/dhcp
```
config dhcp 'tor'
	option interface 'tor'
	option start '100'
	option limit '50'
	option leasetime '1h'
```

Configure the Wireless,

/etc/conf/wireless
```
config wifi-device 'wlan0'
	option type 'mac80211'
	option macaddr 'XX:XX:XX:XX:XX:XX'
	option hwmode '11ng'
	list ht_capab 'LDPC'
	list ht_capab 'SHORT-GI-20'
	list ht_capab 'SHORT-GI-40'
	list ht_capab 'TX-STBC'
	list ht_capab 'RX-STBC1'
	list ht_capab 'DSSS_CCK-40'
	option country 'XX'
	option txpower '20'
	option channel 'XX'
	option htmode 'XX'

config wifi-iface # Private WiFi
	option device 'wlan0'
	option network 'lan'
	option mode 'ap'
	option ssid 'Private'
	option encryption 'psk2+ccmp'
	option key 'XXXXXX'

config wifi-iface # Public WiFi
	option device 'wlan0'
	option network 'tor'
	option mode 'ap'
	option ssid 'The Onion Hotspot'
	option encryption 'none'
	option isolate '1' # Prevent connections between clients
```

Now clients can connect to the SSID "The Onion Hotspot", the router will give them an IP 192.166.2.[100-149] but they couldn't surf the web.

Configure the Firewall,

/etc/config/firewall
```
config zone
	option name 'tor'
	option input 'REJECT'
	option output 'ACCEPT'
	option forward 'REJECT'
	option syn_flood '1'
	option conntrack '1'

config rule
	option name 'Tor DHCP'
	option src 'tor'
	option proto 'udp'
	option target 'ACCEPT'
	option dest_port '67'

config rule
	option name 'Tor Transparent Proxy'
	option src 'tor'
	option proto 'tcp'
	option dest_port '9040' # see /etc/tor/torrc
	option target 'ACCEPT'
	option dest_ip '192.168.2.1'

config rule
	option name 'Tor DNS Proxy'
	option src 'tor'
	option proto 'udp'
	option dest_port '9053' # see /etc/tor/torrc
	option target 'ACCEPT'
	option dest_ip '192.168.2.1'

config redirect
	option target 'DNAT'
	option name 'Tor Middle Relay ORPort'
	option src 'wan'
	option proto 'tcp'
	option src_dport '9001' # see /etc/tor/torrc
	option dest_ip '192.168.2.1'
	option dest_port '9001'
	option dest 'lan'

config redirect
	option target 'DNAT'
	option name 'Tor Middle Relay DirPort'
	option src 'wan'
	option proto 'tcp'
	option src_dport '9030' # see /etc/tor/torrc
	option dest_ip '192.168.2.1'
	option dest_port '9030'
	option dest 'lan'
```

/etc/firewall.user
```bash
iptables -t nat -A PREROUTING -i wlan0-1 -p udp --dport 53 -j REDIRECT --to-port 9053
iptables -t nat -A PREROUTING -i wlan0-1 -p tcp ! --dport 53 --syn -j REDIRECT --to-port 9040
```

Configure USB,

/etc/config/fstab
```
config global 'automount'
	option from_fstab '1'
	option anon_mount '1'

config global 'autoswap'
	option from_fstab '1'
	option anon_swap '0'

config mount
	option target '/mnt/usb'
	option device '/dev/sda1'
	option fstype 'ext4'
	option options 'rw,sync'
	option enabled '1'
	option enabled_fsck '1'

config swap
	option device '/dev/sda2'
	option enabled '1'
```

Run the following terminal commmands.
```bash
$ mkdir -p /mnt/usb
$ mount -a
$ mkdir -p /mnt/usb/tor/{pid,log,geoip}
```

Configure Tor <br/>
First generate the hashed control password, run the following command and copy the output.
```bash
$ tor --hash-password 'MyPassword'
```

/etc/tor/torrc
```
User tor
RunAsDaemon 1
PidFile /mnt/usb/tor/pid/tor.pid
DataDirectory /mnt/usb/tor

# Enable entry and middle relay
Nickname XXXXXX
ContactInfo XXXXXX
ORPort 9001
DirPort 9030
#BridgeRelay 1
Exitpolicy reject *:* # disable exit relay

# Control port
ControlPort 9051
ControlListenAddress 192.168.2.1
HashedControlPassword [[[Paste the hashed password]]]

# Entry Relay
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1                                              
SocksPort 8118 # Enable proxy socks only accesible by the private network 
SocksListenAddress 192.168.2.1
TransPort 9040                                                          
TransListenAddress 192.168.2.1                                          
DNSPort 9053                                                              
DNSListenAddress 192.168.2.1

# Limit middle relay shared bandwidth
RelayBandwidthRate 100 KBytes
RelayBandwidthBurst 200 KBytes

# GeoIP for stats       
# DO NOT UNCOMMENT THIS LINE UNTIL GEOIP SUPPORT IS CONFIRMED  
GeoIPFile /mnt/usb/tor/geoip/geoip
GeoIPv6File /mnt/usb/tor/geoip/geoip6
# Logging:
Log notice file /mnt/usb/tor/log/notice.log
Log notice syslog
# Log debug file /var/log/tor-debug.log

# Tweaks
# Try for at most NUM seconds when building circuits. If the circuit isn't open in that time, give up on it. (Default: 1 minute.)
CircuitBuildTimeout 5
# Send a padding cell every N seconds to keep firewalls from closing our connections while Tor is not in use.
KeepalivePeriod 60
# Force Tor to consider whether to build a new circuit every NUM seconds.
NewCircuitPeriod 15
# How many entry guards should we keep at a time?
NumEntryGuards 8
```

Add GeoIP support
```bash
$ wget http://eurielec.etsit.upm.es/~buker/Tor-GeoIP/geoip -P /mnt/usb/tor/geoip
$ wget http://eurielec.etsit.upm.es/~buker/Tor-GeoIP/geoip6 -P /mnt/usb/tor/geoip
```

We had some troubles with the tor init.d script. In order to fix them we disabled this service and made a custom script.
```bash
$ /etc/init.d/tor disable
```

/etc/rc.local
```bash
if [ -f /mnt/usb/tor/log/notice.log ]
then
    rm /mnt/usb/tor/log/notice.log
fi
sleep 30 && tor &

exit 0
```

Reboot and welcome to the Tor network.<br/<
The router will take at least 5 minutes to connect to the Tor network.

### 2.3 Results ###

After a  week working, these are the stats of the last two days:
```
=>Entry Node: RX: 121.64 MB (1146223 Pkts.) TX: 2.78 GB (2038189 Pkts.)
=>Middle Node: RX: 2.2 GB TX: 4.7GB
```

We made a short survey and these are the conclusions:
Users didn't realize that their traffic is being annonymized but they complained about the bandwidth and the ping of the connection, they felt like surfing through ADSL.

The user experience depends mainly on the congestion of the Tor network: the route made by packets and the server location you want to reach.
The only way to improve the service is speeding up Tor. Adjusting the configuration in order to build better circuits, adding geoip support, improving router hardware (more RAM and ROM).

![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/torimp.png "Tor network with WIFI hotspot")

## 3. Test ##

Tor process occupy from 70MB to 100MB of 128MB available RAM. Without the Tor process running the operating system and network services use 33MB of RAM, and with Tor running it reaches 122MB of RAM. Realizing that we were near a RAM overflow we decided to use part of the USB storage as swap, fortunately after three days up the system used only 8MB of swap. Even if it's useless, we decided to keep swap as a safety measure.

The only traffic allowed on the Tor network is TCP packaged so there is no possibility to run ping or traceroute tests.

After the first test we noticed that the bridge relay only spends up to 80\% of the CPU resources and the bottle neck of the WIFI hotspot is located on the Tor network. In most of the cases, the router handled quite well the WIFI hotspot.

We compared the speed of WIFI connection VS Ethernet connection towards Tor, and we obtained the same values, so we can see that WiFi can afford easily the Tor maximum bandwidth. 

Futhermore we haven't noticed big differences between the connection to the Tor entry node through transparent proxy and proxy socks. That assures a good implementation of the system.

You can see different speed test with iperf here: [Speedtest](https://github.com/SuperBuker/tor-openwrt/blob/master/speedtest/Test.ods)

Starting:
![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/starting.png "CPU and Memory test: Starting")

Idle:
![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/starting.png "CPU and Memory test: Idle")

### 3.1 Middle relay bandwidth ###

In our test time (4 days), the middle relay didn't had a relevant cpu uses even if this process requires encryption/decryption of data passing through the relay because our bandwidth limits were very conservatives. Most of time the process was idle and didn't employed cpu cycles because this middle relay had a very limited bandwidth compared to high speed relays.

The bandwidth configuration is the following:
```
limit: 800.0 Kb/s, burst: 1.5 Mb/s
```

The amount of data transmmited in 4 days:
```
Stats: Download 548.4MB, Upload 547.2MB
```

![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/circuithard.png "Middle relay circuits")

### 3.2 WIFI hotspot capacity ###

At present our WiFi isn't encriped because it's purpuse is to be a public WiFi. The only safety measure we have taken is to enable AP isolation, this prevents connections between WiFi clients and possible direct attacks to our users device.
The connections between the entry relay (our router) and the guard relays chosen are encrypted. 
Speed Test reports 0,5-6Mbps download and upload bandwidth depending on the congestion on the Tor network.
After the tests stats of our public WiFi were the following:
```
RX: 270.72 MB (1120443 Pkts.) TX: 952.88 MB (1319089 Pkts.)
```
WiFi: Maximum Speed
![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/starting.png "WIFI max speed")

Download test: 275KB/s
![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/down275.png "Download test: 275KB/s")

Download test: 450KB/s
![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/down450.png "Download test: 450KB/s")

Download test: 275KB/s
![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/down600.png "Download test: 600KB/s")

## 4. Conclusion ##

When we decided to do this project, we didn't know if it was possible to use a commercial router to do all of these. We knew someone did it before, but they didn't implement both systems together. After our work and testing, we can affirm that is absolutely possible to implement it, the only requeriments is to have a "good" router and compatible with OpenWRT.

Futhermore this implementation may increase easily the number of middle relays of the Tor network, improving network stability, security and troughtput.

The system works as expected. The bandwidth still limited by the Tor network and the hardware isn't a limitation. Sometimes the Tor network hangs and you have to wait until the router rebuilds the circuits, that's why we have improved the circuits rebuild.

Concerning the security issues, Tor had some problems in the past. Someone tried to decipher the communication through our exit node and send it to your destination. In so doing, we can see all data communication. We cannot know who sent the now transparent communication, we only know that we received it from an intermediate TOR node. We can argue that by protecting the communication with the target (eg our bank or our e-mail) with SSL, there is no way for us to get to know the contents of our communications. A security researcher Moxie Marlinspike showed at the last  BlackHat Europe, we can mount a man-in-the-middle attack from our beloved TOR exit server and crack user's SSL communications. <br/>
Moxie gave a real-time demonstration of this attack that obtained a large number of passwords for all kinds of services from many users whose traffic was going through his TOR node.  <br/>
So we can see that the very nature of decentralised and distributed communication on the TOR network brings some problems.

## 4.1 "Wifi hotspot with Tor" Security issues ##

The WiFi hotspot is not encrypted because it's the only way to offer a public WiFi.
Users of this kind of WiFis should encrypt their comunications using https protocol or a VPN, at this time there is no solution to this issue on the network provider side.

In order to make this communication "more" private, we've tried to disable "wifi connections logs" on the router, but actually this didn't provide more security to the users, so we recommend the users that want to get more security to use other way to protect the communications.

When we talked about this with our project supervisor, he suggested us to develop a "tor hotspot client software" that would be able to encrypt the PC-Wifi hotspot communication.

The main problem of any encrypting solution is that they is necessarily an encrypting key/passphrase exchange, this should be done before we cypher the connection. Assuming that there isn't any users registry and in consecuence any possibility of sharing the key before the start of the connection, this should be done just after the connecting request and this packets can be intercepted. In conclusion the connection can be monitorized and decrypted in real time and there is any advantage over a non encrypted network.

![alt text](https://github.com/SuperBuker/tor-openwrt/raw/master/doc/img/torsecurity.png "Tor network with WIFI hotspot security problem")

### 4.2 Middle relay implementation and viability ###

The implementation of the middle relay is quite simple if you have already set up a Tor Hotspot. It needs much more ram than a simple Tor client but the advantage is that your traffic is mixed with the relays one.

A middle relay doesn't have security problem but the configuration is a bit complicated and you could leave a backdoor to your network and that is dangerous.

The more middle relays we have, the better works Tor network does and more secure will be.
