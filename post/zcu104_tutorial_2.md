## Connect PC and ZCu104 directly using ethernet cable

1. Connect the ZCU104 board and PC with ethernet cable. **BECAREFUL** use ethernet card, not USB_ETH adapter
2. Configure the ip address of ZCU104 by the following command **vi /etc/network/interfaces**, and replace the line 
"iface eth0 inet dhcp" with the following setting
iface eth0 inet static
	address 192.168.1.22
	netmask 255.255.255.0
	network 192.168.1.0
	gateway 192.168.1.11
3. Set the ip address of host PC as following
auto <YOUR ETH CARD>
iface <YOUR ETH CARD> inet static
    address 192.168.1.11
    netmask 255.255.255.0
4. Ping the ip adress of each other

