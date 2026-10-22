# WAN Network
Network → Networks → Create Network

 **Network tab**
| Field | Value |
|---|---|
| Network Name | `wan-net` |
| Enable Admin State | checked |
| Create Subnet | checked |
| Availability Zone Hints | leave default (`nova`) |
| MTU | leave blank


**Subnet tab**
| Field | Value |
|---|---|
| Subnet Name | `wan-subnet` |
| Network Address | `10.10.0.0/24`* |
| IP Version | IPv4 |
| Gateway IP | leave blank (becomes `10.10.0.1`) |
| Disable Gateway | unchecked |

*you can use whatever IP you want. Just make sure it's unique and take note of the mask


**Subnet Details tab**
| Field | Value |
|---|---|
| Enable DHCP |checked |
| Allocation Pools | leave blank |
| DNS Name Servers | `1.1.1.1` and `8.8.8.8` (one per line) |
| Host Routes | leave blank |

Click **Create**

# LAN Network
**Network → Networks → Create Network**

**Network tab**
| Field | Value |
|---|---|
| Network Name | `lan-net` |
| Enable Admin State | checked |
| Create Subnet | checked |
| Availability Zone Hints | leave default |
| MTU | leave blank |


**Subnet tab**
| Field | Value |
|---|---|
| Subnet Name | `lan-subnet` |
| Network Address | `192.168.50.0/24`* |
| IP Version | IPv4 |
| Gateway IP | leave blank |
| Disable Gateway | **checked** (pfSense will be the gateway) |

*you can use whatever IP you want. Just make sure it's unique and take note of the mask


**Subnet Details tab**
| Field | Value |
|---|---|
| Enable DHCP | **unchecked** (pfSense will run DHCP) |
| Allocation Pools | leave blank |
| DNS Name Servers | leave blank |
| Host Routes | leave blank |

Click **Create**
# OpenStack Router
**Network → Routers → Create Router**

| Field | Value |
|---|---|
| Router Name | `edge` |
| Enable Admin State |  checked |
| External Network | **External Network** (HackUCF's; open the dropdown to confirm it's actually selected, not placeholder text) |
| Availability Zone Hints | leave blank |

Click **Create Router**


Then click **edge → Interfaces tab → Add Interface**:
| Field | Value |
|---|---|
| Subnet | `wan-net: 10.10.0.0/24 (wan-subnet)` |
| IP Address | leave blank |


# pfSense Ports
**Network → Networks → lan-net → Ports tab → Create Port**

| Field | Value |
|---|---|
| Name | `pf-lan` |
| Enable Admin State | checked |
| Device ID | blank |
| Device Owner | blank |
| Specify IP address or subnet | **Fixed IP Address** → `192.168.50.1` * |
| MAC Address | blank |
| Port Security | **unchecked** |
| VNIC Type | Normal |

Click **Create**. The port will show **Detached / Down** until it's attached to the VM
*use whatever IP you set earlier: `X.X.X.1`

# Launch Router
**Compute → Instances → Launch Instance**

| Tab | Setting |
|---|---|
| Details | Instance Name: `pfsense` |
| Source | Boot Source: **Image** → `pfSense 2.7.2`; Create New Volume: **Yes** (persistent disk) |
| Flavor | `m1.small` (1 vCPU, 1 GB RAM) or larger |
| Networks | **`wan-net` only** |
| Network Ports | leave empty |
| Security Groups | leave empty |
| Key Pair | your default key |

Click **Launch Instance**
*The order matters: the network present at boot becomes **vtnet0 (WAN)**; the port attached next becomes **vtnet1 (LAN)**.*

# Attach the LAN port and fix the WAN port
1. **Attach LAN:** Instance dropdown → **Attach Interface**
   - The way to specify an interface: **by Port**
   - Port: `pf-lan`
2. **Disable port security on the auto-created WAN port:**
   - **Network → Networks → wan-net → Ports**, find the port whose Attached Device is the pfSense instance (not `pf-wan`). Should say ` compute:nova` in the "Attached Device" column
   - **Edit Port → Security Groups tab** → remove all groups → Save
   - **Edit Port** again → uncheck **Port Security** → Save
3. Delete the unused `pf-wan` port if one exists

**Verify:** the instance's Overview page should list an IP on both `wan-net` (10.10.0.x) and `lan-net` (192.168.50.1), and **Security Groups: Not available** (meaning port security is off on both ports)

# Router Config

Open the instance's **Console** tab.
Because `pf-lan` was attached after boot, pfSense will report **"Network interface mismatch"** and start interface assignment (it may only list `vtnet0` at first). 

You can go through the setup by pressing enter if it prompts anything. Try to get to the main menu screen. Be patient!

At the console menu, choose **2 Set interface(s) IP address** → select **LAN**:
| Prompt | Answer |
|---|---|
| Configure IPv4 address via DHCP? | `n` |
| New LAN IPv4 address | `192.168.50.1`* |
| Subnet bit count | `24`* |
| Upstream gateway | press **Enter** (none) |
| Configure IPv6 via DHCP6? | `n` |
| New LAN IPv6 address | press **Enter** (none) |
| Enable DHCP server on LAN? | `y` |
| Start address of range | `192.168.50.100`* |
| End address of range | `192.168.50.200`* |
| Revert to HTTP for webConfigurator? | `n` |

*use whatever IP/mask you set earlier

The menu header should now show:
```
WAN (wan) -> vtnet0 -> v4/DHCP4: 10.10.0.210/24
LAN (lan) -> vtnet1 -> v4: 192.168.50.1/24
```

# Assign Floating IP
In Horizon: instance dropdown → **Associate Floating IP** → choose the WAN port

The instance Overview will show the floating IP next to the WAN address 

Browse to `https://<floating-ip>` from your desktop. Default login: **admin / pfsense**

