# Networking

## Routing, Switching, Gateways

To find gateway:

```shell
route
# or
ip route list
```

To add entries into the routing table. Where 2nd ip is gateway

```shell
ip route add 192.168.1.0/24 via 192.168.2.1
```



If forwarding between machines required for communication without a router:

`/proc/sys/net/ipv4/ip_forward` must be enabled:

```shell
echo 1 > /proc/sys/net/ipv4/ip_forward
```

To persist:

```
# /etc/sysctl.conf
net.ipv4.ip_forward=1
```

## DNS

In `/etc/resolv.conf` set DNS server:

```
nameserver 192.168.1.100
```

We can set order for `/etc/hosts` or DNS server in:

`/etc/nsswitch.conf`:

```
hosts:  files dns
```

We can use `nslookup`, `dig` to query DNS servers:

```
nslookup google.ca
dig google.ca
```

## Network Namespaces

Lets us have isolated routing and arp tables along with virtual interfaces.

```shell
ip netns add b
ip netns list
```

Run in ns:

```shell
ip netns exec red ip link
# or
ip -n res link
```

To connect namespaces we can use a virtual pair (or pipe):

To create a virtual cable

```shell
ip link add veth-red type veth peer name veth-blue
```

To attach with the network namespaces

```shell
ip link set veth-red netns red
ip link set veth-blue netns blue
```

To add an IP address

```shell
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blue
```

To set up `ns` interfaces

```shell
ip -n red link set veth-red up
ip -n blue link set veth-blue up
```

Check the connectivity

```shell
ip netns exec red ping 192.168.15.2
```

When we have many NS, we create a switch (bridge)

---

Putting this all together we can have the bridge reach the external network by talking to our host as the Gateway, and have connections go back in to the private network by implementing NAT on our host via:


```shell
iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
```

## Pod Networking

The rules of kubernetes pod networking are that:

* every pod should have an IP address
* every pod should be able to communicate with every other pod in the same node
* every pod should be able to communicate with every other pod on other nodes without NAT

We create a bridge on each node for the containers.
Each bridge has a private subnet.
To allow cross-node communications we add routes between nodes or use a router.

See `kube-controller-manager` `--cluster-cidr=` for pod range.

## CNI in Kubernetes

We specify CNI plugin on container runtime in `/etc/cni/net.d`, bins in `/opt/cni/bin`

Kubernetes networking Solutions will typically install agents on every node (DaemonSet) along with bridges and then deal with peer-to-peer communication


## IP Address Managements (IPAM)

Who assigns IPs to containers.
CNI plugin manages the IP management.

## Service Networking

Pods communicate via services and each gets a cluster-wide IP.

kube-proxy watches for service creation, and creates one.
This is done by each node setting up forwarding rules on each node.

proxy-mode defines how forwarding rules are created on kube-proxy.

`service-cluster-ip-range` defines ip range for services.

```shell
ps -aux | grep kube-apiserver
```

```
--secure-port=6443 --service-account-key-file=/etc/kubernetes/pki/sa.pub --
service-cluster-ip-range=10.96.0.0/12
```

## DNS in Kubernetes

Whenever we create services they get a DNS entry so any pod can access.

If in same namespace, can use just service name, eg 'service'

In different namespace add namespace suffix, eg 'service.default'.

Full domain is 'service.default.svc' or FQDN 'service.default.svc.cluster.local'

By default pods do not get entry, but we can enable DNS enties for them, thry get entry:

'IP-WITH-DASHES.namespace.pod'

eg '10-244-2-5.default.pod'


