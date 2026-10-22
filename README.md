# Intro
This lab contains the instructions for setting up a virtural home lab on Hack@UCF's club infrastructurte. Please complete the user guides "Overview of OpenStack GUI" through "How to Launch an Instance in OpenStack GUI" before this lab (https://help.hackucf.org/)

# Sub-Labs
The plan is to add more folders to this repo containing more speciailized labs (eg routing, ISD/IPS, etc).


# Network Map

   ![HackUCF "External Network"   (shared, unremovable — provides internet + floating IPs)
        │
   [edge]  OpenStack router  (external gateway + one interface on wan-net)
        │
   wan-net  (OpenStack DHCP on, gateway enabled)
        │
        │  vtnet0 = pfSense WAN  (DHCP + floating IP)
   [ pfSense VM ]
        │  vtnet1 = pfSense LAN  (static, pfSense DHCP server)
        │
   lan-net (OpenStack DHCP OFF, no OpenStack gateway)
        │
   lab VMs (get IP from pfSense)
](image.png)