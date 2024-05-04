# wireguard_instruction
Instruction for configuration VPN using Wireguard in Terminal.

SERVER CONFIGURATION
1) You need to rent server. (If you live in Russia, you need to choose foreign host. Personally, I used "ishosting" - is Estonian host.)
2) Then, you need to configure server. You need to open terminal and write "ssh root@<IPaddress_of_your_server>" and enter the password of your server.
3) You need to download wireguard on your server. Write "apt install wireguard". Also, you need to donwload iptables package - "apt install iptables".
4) Now, you need to generate private and public key, using command "wg genkey | tee /etc/wireguard/server_private_key | wg pubkey | tee /etc/wireguard/server_public_key" and write "chmod 600 /etc/wireguard/server_private_key"
5) You need to get network interface on your server, write "ip a | grep "mtu 1500", most probably is "eth0".
6) Now, you need to make config file of your server "nano /etc/wireguard/wg0.conf" with this text:

[Interface]
Address = 10.0.0.1/24
ListenPort = 51830
PrivateKey = server_private_key                                                                       //insert ypur info
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o
eth0 -j MASQUERADE                                                                                    //if you dont have eth0, change this parameter
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o
eth0 -j MASQUERADE

7) You need to allow IPv4 forwarding "echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p" .
8) To activate you need to write "wg-quick up wg0" and "wg-quick down wg0" to disactivate.

CLIENT CONFIGURATION
1) Now, you need to download wireguard, open new window in terminal and generate clinet keys "wg genkey | sudo tee /etc/wireguard/client_private_key | sudo wg pubkey | sudo tee /etc/wireguard/client_public_key"
2) Then, you need to add following text to server config file and restart it.

[Peer]
PublicKey = client_public_key                                                                          //insert your info
AllowedIPs = 10.0.0.2/32

4) Then, create wg0-client.conf in /etc/wireguard/ and insert this text:

[Interface]
PrivateKey = client_private_key                                                                       //insert your info
Address = 10.0.0.2/32
DNS = 8.8.8.8
[Peer]
PublicKey = server_public_key                                                                         //insert your info
Endpoint = IP_SERVER:51830                                                                            //insert your info
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20

6) 
wg-quick up wg0-client   - turn on the vpn

wg-quick down wg0-client - turn off the vpn

8) Furthermore, you need to make routes:

ip route add IP_SERVER/32 via YOUR_IP
ip route add 0.0.0.0/1 via 10.0.0.1
ip route add 128.0.0.0/1 via 10.0.0.1

To return your default ip you need to delete routes.

8) To check connection try:
   ping 10.0.0.1
   traceroute -m1 -n 1.1.1.1
   ping 1.1.1.1

   if you can see good transmission of packets it means your vpn works correctly.

Additional info:
- some sites have top priority with IPv6 and we made only IPv4 route, you can try to delete IPv6 "ip addr del <IPv6>" or change priority in file on your PC.
- browser can save cache about your IP and because of it your VPN doesn`t work right way, you should reboot browser or use another.
