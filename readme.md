# Self Host VPN with WireGuard
### What is a VPN?

VPN stands for **Virtual Private Network**.

A VPN allows you to securely connect to your internal (private) network from outside the network over the internet.

---

### Why is a VPN needed in a Home Lab?

A VPN allows you to securely configure, troubleshoot, and fix your home lab services even when you are away from home.

---

### What is WireGuard?

WireGuard is an **open-source**, lightweight, and secure VPN application that uses modern encryption to create fast and reliable VPN connections.

----
### Directory Structure

```
cd /opt/docker_files
mkdir wireguard
mkdir wireguard/data
```

```
 $ tree /opt/docker_files/wireguard/
/opt/docker_files/wireguard/
├── data
│   └── wireguard
└── docker-compose.yml
```
-----------------
## Download docker-compose.yml

```
wget
```
----

