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
### Download docker-compose.yml

```
git clone https://github.com/TejaNune1998/WireGuard_Stack.git
```

----
### Directory Structure
```
$ tree /opt/docker_files/wireguard/
/opt/docker_files/wireguard/
├── data
│   └── wireguard
└── docker-compose.yml
```

---------
### Update docker-compose.yml

* Update environment 
```   
environment:
      - WG_HOST=vpn.</mydomain.com>
      - PASSWORD=</passwd>
      - WG_DEFAULT_DNS=</dns1>, </dns2>
    
      # **comment/uncomment which is requried.**
      # Split Tunnel (ONLY internal network through VPN)
      #- WG_ALLOWED_IPS=</internal_network>
      # Full Tunnel (ALL traffic including internet through VPN)
      - WG_ALLOWED_IPS=</internal_network>, 0.0.0.0/0   
```

* Update networks 
```
services:
  wg-easy:
	networks:
      - </traefik_network>
```

```
networks:
  </traefik_network>:
    external: true
```

* Update Labels
```
    labels:
      - "traefik.http.routers.wireguard-secure.tls.certresolver=<your_certresolver>"
      - "traefik.http.routers.wireguard.rule=Host(`wireguard.</mydomain.com>`)"
      - "traefik.http.routers.wireguard-secure.rule=Host(`wireguard.</mydomain.com>`)"
      - "traefik.docker.network=</traefik_network>"
```
--------------
### Create a CNAME record in Pihole (DNS) for Web UI

1. Login to Pi-hole dashboard
```
https://pi-hole.</mydomain.com>/admin
```
2. Navigate to **Settings → Local DNS Settings  → Local CNAME records**
3. Create CNAME record as below and click "+" to add

| Domain                    | Target                     |
| ------------------------- | -------------------------- |
| wireguard.</mydomain.com> | dockinator.</mydomain.com> |


*dockinator.</domain.com> :  is docker host where Traefik is running.*

-----------------
### Start our compose stack as a daemon.

```
docker compose up -d
```
---
### Get Public IP details

```
curl ifconfig.me
```
----------------
### Create A record in Cloudflare (Domain Provider) 

1. Login to Cloudflare.
```
https://dash.cloudflare.com
```
2. Select respective domain.
3. Navigate to **DNS → Records** and click **Add record**
4. Create record as below and click Save.

| Type | Name | IPv4 Address  | Proxy status       | TTL  |
| ---- | ---- | ------------- | ------------------ | ---- |
| A    | vpn  | </Public_IP/> | DNS only (uncheck) | Auto |

---
### Create Firewall Rules
*  Login to Sohpos admin UI
```
https://</sophos_ip_or_hostname>:4444
```
* Create IP Host for WireGuard Container docker host.
	* Navigate to **System → Host and services → IP Host**.
	* Click **Add**.
	* Create as below:
		* Name : </Hostname -  Container Host>
		* IP version : IPv4
		* Type : IP
		* IP address : </IP -  Container Host>
	* Click Save.

* Create IP Host for VPN clients.
	* Navigate to **System → Host and services → IP Host**.
	* Click **Add**.
	* Create as below:
		* Name : WireGuard_VPN_Clients
		* IP version : IPv4
		* Type : Network
		* IP address : 10.8.0.0/24

* Create IP Host for Internal Network.
	* Navigate to **System → Host and services → IP Host**.
	* Click **Add**.
	* Create as below:
		* Name : Internal_Network
		* IP version : IPv4
		* Type : Network
		* IP address : 10.1.0.0/16 (internal network)

* Create Service
	* Navigate to **System → Host and services → Services**.
	* Click **Add**.
	* Create as below:
		* Name : WireGuard
		* Type: TCP/UDP
		* Protocol: UDP
		* Source port: 1:65535
		* Destination port: 51820
	* Click Save.

* Create DNAT Rule
	* Navigate to **Protect → Rule and policies →  Firewall rules.**
	* Click Add **firewall rule** and select **Server access assistant (DNAT)**.
	* Create as below
		* Internal server IP address : *choose newly created docker host IP Host from drop down.*
		* Public IP Address : choose WAN port
		* Services : **Click Add new item** and choose newly created service
		* External source networks and devices : Any
		* Review and click **Save and finish**.

* Create rules for VPN network to Internal network
	1. Allow VPN to Internal Network
		* Navigate to **Protect → Rule and policies →  Firewall rules.**
		* Click Add **firewall rule** and select **New firewall rule**.
		* Create as below
			* Name : Allow VPN to Internal Network
			* Action : Accept
			* Source zones : LAN
			* Source networks and devices : WireGuard_VPN_Clients
			* Destination zones : LAN
			* Destination networks : Any
			* Services : Any

	2. Allow WireGuard VPN
		* Navigate to **Protect → Rule and policies →  Firewall rules.**
		* Click Add **firewall rule** and select **New firewall rule**.
		* Create as below
			* Name : Allow WireGuard VPN
			* Action : Accept
			* Source zones : WAN
			* Source networks and devices : Any
			* Destination zones : LAN
			* Destination networks : *choose newly created docker host IP Host from drop down.*
			* Services : WireGuard

* Create a rule for access VPN clients to internet while connected to VPN
	* Navigate to **Protect → Rule and policies →  Firewall rules.**
		* Click Add **firewall rule** and select **New firewall rule**.
		* Create as below
			* Name : Allow VPN to Internet
			* Action : Accept
			* Source zones : LAN
			* Source networks and devices : WireGuard_VPN_Clients
			* Destination zones : WAN
			* Destination networks : Any
			* Services : Any

* Create Static Route
	* Navigate to **Configure → Routing → Static routes**.
	* Under **IPv4 unicast route**.
	* Click add and create as below
		* Destination IP / Netmask : 10.8.0.0/24
		* Gateway : </IP - Docker Host>
		* Interface : </ LAN Port>
		* Administrative distance : 1
	* Click Save.
---------
### Create VPN profile for user
* Login to wireguard web ui using **Password** in docker-compose.yml
```
https://wireguard.</domain.com>
```
* Click on "+ New" and provide the Name.
-----------------
### Download WireGuard client tools

```
https://www.wireguard.com/install
```

----------------
### Install VPN user profile in client tool

* Login to wireguard web ui.
```
https://wireguard.</domain.com>
```

* For Windows
	* Click on **download icon** in Web page.
	* Open the WireGuard application in Windows
	* Click on **import**.
	* Use **Activate** to connect to VPN and **Deactivate** to disconnect from VPN.

* For IOS
	* Click on **QR code icon** in Web page.
	* Open WireGuard App in IOS device.
	* Click "+ icon" in IOS device.
	* Select **Create from QR code** and scan the QR code
	* Give a name.
	* Use the toggle switch to connect and disconnect to VPN.
