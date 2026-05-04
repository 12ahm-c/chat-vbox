┌──(xendi㉿shadow)-[~]
└─$ sudo wg-quick up wg0          
[#] ip link add dev wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 192.168.8.153/32 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] resolvconf -a tun.wg0 -m 0 -x
[#] ip -4 route add 192.168.9.0/24 dev wg0
[#] ip -4 route add 10.10.7.0/24 dev wg0
                                                                                                                                                                        
┌──(xendi㉿shadow)-[~]
└─$ 
