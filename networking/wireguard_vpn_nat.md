# wireguard_vpn_nat

## /etc/wireguard/wg0.conf

to create interface wg0 on my.server

```
[Interface]
Address = 172.30.1.1/24
ListenPort = 6767
PrivateKey = $(wg genkey)

# NAT for clients
PostUp = iptables -I FORWARD -i wg0 -j ACCEPT; iptables -I FORWARD -o wg0 -j ACCEPT ; iptables -I FORWARD -i wg0 -j ACCEPT ; iptables -t nat -I POSTROUTING -o enp0s6 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT ; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s6 -j MASQUERADE

# CLIENT 1
[Peer]
PublicKey = $(echo CLIENT1_PrivateKey | wg pubkey)
AllowedIPs = 172.30.1.11/32

# CLIENT 2
[Peer]
PublicKey = $(echo CLIENT2_PrivateKey | wg pubkey)
AllowedIPs = 172.30.1.12/32

```

### /etc/wireguard/clients/client1.conf

to generate qr code for mobile

```
[Interface]
PrivateKey = $(wg genkey)
Address = 172.30.1.11/24
DNS = 8.8.8.8

[Peer]
PublicKey =  $(echo SERVER_Pricatekey | wg pubkey)
# all traffic
AllowedIPs = 0.0.0.0/0
Endpoint = my.server:6767
PersistentKeepalive = 15

# qrencode -t ansiutf8 < client1.conf
```

### /etc/wireguard/clients/client2.conf

to import for a wireguard client

```
[Interface]
PrivateKey = $(wg genkey)
Address = 172.30.1.12/24
DNS = 8.8.8.8

[Peer]
PublicKey =  $(echo SERVER_Pricatekey | wg pubkey)
AllowedIPs = 172.30.1.0/24
Endpoint = my.server:6767
PersistentKeepalive = 15
```

## commands

```
apt install wireguard nload qrencode 

# start and enable wg0.conf at boot 
systemctl enable --now wg-quick@wg0

# manual control 
# wg-quick up wg0 
# wg-quick down wg0 

# hard restart (stops and starts - PARSE ERROR = STOP)
# systemctl restart wg-quick@wg0

# reload
wg syncconf wg0 <(wg-quick strip wg0)

# check connection status and client speedtest
# watch -n1 wg show
# nload
```
