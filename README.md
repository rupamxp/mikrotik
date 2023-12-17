DUAL WAN LOAD Balance Configuration In Mikrotik ROUTER

/ip address
add address=192.168.0.1/24 network=192.168.0.0 broadcast=192.168.0.255 interface=Local
add address=192.168.1.2/24 network=192.168.1.0 broadcast=192.168.1.255 interface=WAN1
add address=192.168.2.2/24 network=192.168.2.0 broadcast=192.168.2.255 interface=WAN2

/ip dns set allow-remote-requests=yes cache-max-ttl=1w cache-size=5000KiB max-udp-packet-size=512 servers=221.132.112.8,8.8.8.8

/ip firewall mangle
add chain=input in-interface=WAN1 action=mark-connection new-connection-mark=WAN1_conn
add chain=input in-interface=WAN2 action=mark-connection new-connection-mark=WAN2_conn

add chain=output connection-mark=WAN1_conn action=mark-routing new-routing-mark=to_WAN1
add chain=output connection-mark=WAN2_conn action=mark-routing new-routing-mark=to_WAN2

add chain=prerouting dst-address=192.168.1.0/24 action=accept in-interface=Local
add chain=prerouting dst-address=192.168.2.0/24 action=accept in-interface=Local

add chain=prerouting dst-address-type=!local in-interface=Local per-connection-classifier=both-addresses-and-ports:2/0 action=mark-connection new-connection-mark=WAN1_conn passthrough=yes
add chain=prerouting dst-address-type=!local in-interface=Local per-connection-classifier=both-addresses-and-ports:2/1 action=mark-connection new-connection-mark=WAN2_conn passthrough=yes

add chain=prerouting connection-mark=WAN1_conn in-interface=Local action=mark-routing new-routing-mark=to_WAN1
add chain=prerouting connection-mark=WAN2_conn in-interface=Local action=mark-routing new-routing-mark=to_WAN2

/ip route
add dst-address=0.0.0.0/0 gateway=192.168.1.1 routing-mark=to_WAN1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.2.1 routing-mark=to_WAN2 check-gateway=ping

add dst-address=0.0.0.0/0 gateway=192.168.1.1 distance=1 check-gateway=ping
add dst-address=0.0.0.0/0 gateway=192.168.2.1 distance=2 check-gateway=ping

/ip firewall nat
add chain=srcnat out-interface=WAN1 action=masquerade
add chain=srcnat out-interface=WAN2 action=masquerade
