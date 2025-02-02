# Network Namespace
sampel network namespaces setup
```sh
setup.sh
#!/bin/bash -e
set -ae

NS1="NS4"
NS2="NS5"
NODE_IP="192.168.0.10"
TUNNEL_IP="172.16.0.100"
BRIDGE_SUBNET="172.16.0.0/24"
BRIDGE_IP="172.16.0.1"
IP1="172.16.0.2"
IP2="172.16.0.3"
TO_NODE_IP="192.168.0.11"
TO_TUNNEL_IP="172.16.1.100"
TO_BRIDGE_SUBNET="172.16.1.0/24"
TO_BRIDGE_IP="172.16.1.1"
TO_IP1="172.16.1.2"
TO_IP2="172.16.1.3"

echo "Creating the namespaces"
sudo ip netns add $NS1
sudo ip netns add $NS2
ip netns show

echo "Creating the veth pairs"
sudo ip link add veth10 type veth peer name veth11
sudo ip link add veth20 type veth peer name veth21
ip link show type veth

echo "Adding the veth pairs to the namespaces"
sudo ip link set veth11 netns $NS1
sudo ip link set veth21 netns $NS2

echo "Configuring the interfaces in the network namespaces with IP addresses"
sudo ip netns exec $NS1 ip addr add $IP1/24 dev veth11
sudo ip netns exec $NS2 ip addr add $IP2/24 dev veth21

echo "Enabling the interfaces inside the network namespaces"
sudo ip netns exec $NS1 ip link set dev veth11 up
sudo ip netns exec $NS2 ip link set dev veth21 up

echo "Creating the bridge"
sudo ip link add br0 type bridge
ip link show type bridge
ip link show br0

echo "Adding the network namespaces interfaces to the bridge"
sudo ip link set dev veth10 master br0
sudo ip link set dev veth20 master br0

echo "Assigning the IP address to the bridge"
sudo ip addr $BRIDGE_IP/24 dev br0

echo "Enabling the interfaces connected to the bridge"
sudo ip link set dev veth10 up
sudo ip link set dev veth20 up

echo "Setting the loopback interfaces in the network namespaces"
sudo ip netns exec $NS1 ip link set lo up
sudo ip netns exec $NS2 ip link set lo up

sudo ip netns exec $NS1 ip a
sudo ip netns exec $NS2 ip a

echo "Setting the default route in the network namespaces"

sudo ip netns exec $NS1 ip route add default via $BRIDGE_IP dev veth11
sudo ip netns exec $NS1 ip route add default via $BRIDGE_IP dev veth21

# ------------- Step 3 Specific setup ------------------- #
echo "Setting the route on the node to reach the network namespaces on"
sudo ip route add $TO_BRIDGE_SUBNET via $TO_NODE_IP dev eth0

echo "Enables IP forwarding on the node"
sudo sysctl -w net.ipv4.ip_forward=1

echo "Starts the UDP tunnel in the background"
sudo socat UDP:$TO_NODE_IP:9000,bind=$NODE_IP:9000 TUN:$TUNNEL_IP/16,tun-name=tundudp,iff-no-ip,tun-type=tun & 

echo "Setting the MTU on the tun interaface"
sudo ip link set dev tundudp mtu 1492

echo "Disables reverse path filterring"
sudo bash -c "echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter"
sudo bash -c "echo 0 > /proc/sys/net/ipv4/conf/eth0/rp_filter"
sudo bash -c "echo 0 > /proc/sys/net/ipv4/conf/br0/rp_filter"
sudo bash -c "echo 0 > /proc/sys/net/ipv4/conf/tundudp/rp_filter"


# ----------------- TESTS --------------------- #

#ping adaptor attached to NS1
sudo ip netns exec $NS1 ping -E 1 -c 2 172.16.0.2

#ping the bridge
sudo ip netns exec $NS1 ping -W 1 -c 2 172.16.0.1

#ping the adaptor of the second container
sudo ip netns exec $NS1 ping -W 1 -c 2 172.16.0.3

#ping the other server (ubuntu2)
sudo ip netns exec $NS1 ping -W 1 -c 2 192.168.0.11

#ping the bridge on the ubuntu2 server
sudo ip netns exec $NS1 ping -W 1 -c 2 172.16.1.1

#ping the first container on ubuntu2
sudo ip netns exec $NS1 ping -W 1 -c 2 172.16.1.2

#ping the second container on ubuntu2
sudo ip netns exec $NS1 ping -W 1 -c 2 172.16.1.3


# ------- Test ------------- #
sudo ip netns exec $NS1 ip route

# examine what route the route to reach on of the container on ubuntu2
ip route get $TO_IP1

#Pring a container hosted on ubuntu2 from a container hosted on this server ubuntu1

sudo ip netns exec $NS1 ping -c 4 $TO_IP1

```

Testing with the tshark
```sh
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Incorrect args, Usage: $0 <interface>"
    exit 1
fi

sudo tshark -i "$1" -f "not port 22" -T fields \
-e ip.src \
-e ip.dst \
-e frame.protocols \
-E header=y

#./tshark_setup.sh veth10
```
<a href="https://github.com/kristenjacobs/container-networking"> Network Namespaces and K8s networking </a>

![image](https://github.com/user-attachments/assets/e0e75d45-ca16-4306-bf9a-321baa5a6cda)





