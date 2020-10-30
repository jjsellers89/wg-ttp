# wg-ttp
Use WG secure tunneling to bridge remote user into a LAN

############
' Overview '
############

Items used:
- Sys76 (remote user)
- AWS instance with publicly available IP (Debian 10 AMI)
- Pi Zero W (to enable bridging into LAN)
- LAN consisted of 1x Openwrt router w/ 1 attached client (Pi 3 webserver)

Scenario:
- Using WG secure tunneling, route traffic from Sys76, through AWS instance and Pi Zero W to a LAN of interest.
- Pi Zero W has eth0 connected to public internet and wlan0 connected to LAN of interest.
- The following configuration will enable the remote Sys76 laptop to access the LAN of interest via WG secure tunneling.

LAN of interest - 192.168.1.0/24
Openwrt Router = 192.168.1.1/24
Webserver = 192.168.1.202/24

#################################
' Steps to establish WG tunnels '
#################################

(these steps assume you have installed/compiled WG already on the Sys76, AWS instance and Pi Zero W)

--- AWS instance (WG IP: 9.9.9.99/24)

WG: 
wg set wg0 listen-port 12345 private-key private peer <Sys 76 Pub Key> allowed-ips 9.9.9.1/32 peer <Pi Zero W Pub Key> allowed-ips 9.9.9.2/32,192.168.1.0/24

Additional commands: 
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -I POSTROUTING -o wg0 -j MASQUERADE
ip route add 192.168.1.0/24 via 9.9.9.2

--- Sys76 laptop (WG IP: 9.9.9.1/24)

WG:
wg set wg0 private-key private peer <AWS Public Key> endpoint <AWS pub IP>:12345 persistent-keepalive 10 allowed-ips 9.9.9.0/24,192.168.1.0/24

Add'l commands:
ip route add 192.168.1.0/24 via 9.9.9.99 

--- Pi Zero W (WG IP: 9.9.9.2/24)

WG:
wg set wg0 private-key private peer <AWS Public Key> endpoint <AWS Pub IP>:12345 persistent-keepalive 10 allowed-ips 9.9.9.0/24

Add'l commands:
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -I POSTROUTING -o wlan0 -j MASQUERADE

####################
' Success criteria '
####################

- Curl, netcat and/or navigate to OpenWRT router homepage (192.168.1.1) from the Sys76 workstation, as if you were connected directly to network.
- Curl, netcat and/or navigate to LAN webserver (192.168.1.202) from the Sys76 workstation, as if you were connected directly to network.
