gre tunnel

all server-

sudo modprobe ip_gre
lsmod | grep gre

sudo apt install iptables iproute2

-------------------------------------

on tunnel server

sudo echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sudo sysctl -p

sudo ip tunnel add gre1 mode gre local tunnel_ip remote main_ip ttl 255
sudo ip addr add 10.0.0.1/30 dev gre1
sudo ip link set gre1 up

on main server

sudo ip tunnel add gre1 mode gre local main_ip remote tunnel_ip ttl 255  
sudo ip addr add 10.0.0.2/30 dev gre1  
sudo ip link set gre1 up

--------------------------------------

ping test
ping 10.0.0.2
ping 10.0.0.1

-----------------------

on main server

sudo echo '100 GRE' >> /etc/iproute2/rt_tables  
sudo ip rule add from 10.0.0.0/30 table GRE  
sudo ip route add default via 10.0.0.1 table GRE

-------------------------

on tunnel server

iptables -t nat -A POSTROUTING -s 10.0.0.0/30 ! -o gre+ -j SNAT --to-source tunnel_ip

---------------------

on main server

sudo apt install curl
curl https://www.euro-space.net/show-ip
(ip of tunnel server should come)

-------------------------

on tunnel server

sudo iptables -A FORWARD -d 10.0.0.2 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.0.2 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A PREROUTING -d tunnel_ip -p TCP -m TCP --dport 25565 -j DNAT --to-destination 10.0.0.2

note
On a server restart all the things we did will be wiped out. To make sure the GRE tunnel and everything else is going to work after a restart we have to edit the file /etc/rc.local and add all the commands we did (except for the echo ones!) before the exit 0.

Now, if we connect to Server A using the ports we configured (Port TCP 80 for example) we're going instead to connect to Server B without knowing it.

Note: if you use CSF to manage iptables you may have to put all the iptables commands in your /etc/csf/csfpost.sh and insert both servers' IP (also the GRE one) in your /etc/csf/csf.allow.

 
