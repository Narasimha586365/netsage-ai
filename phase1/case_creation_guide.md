# Phase 1 Case Creation Guide

This guide provides reproducible Packet Tracer lab recipes for the 30 records in `cases.csv`. Build the **initial working configuration**, verify it, and then introduce only the stated fault. Use PC Command Prompt for `ipconfig`, `ping`, and `tracert`; use Laptop-PT Desktop/PC Wireless and Server-PT Services pages when named.

## VLAN-01 — PC connected to the wrong access VLAN

1. **Devices required:** PC-PT Fa0 to 2960 Fa0/2; Finance VLAN 10 uses router gateway 192.168.10.1.
2. **Basic topology:** PC-PT Fa0 to 2960 Fa0/2; Finance VLAN 10 uses router gateway 192.168.10.1.
3. **Initial working configuration:** Create the referenced VLAN(s) on each required 2960, configure PC-facing ports as access ports in their intended VLAN, and use a trunk for any switch-to-switch link. Configure the stated gateway if routing is involved.
4. **Fault to intentionally introduce:** Switchport Fa0/2 is assigned to VLAN 20 instead of VLAN 10.
5. **Expected symptom:** Finance PC cannot reach its Finance gateway or peers; it receives/uses an address for another segment.
6. **Commands to collect evidence:** `S1# show vlan brief; S1# show running-config interface fa0/2; PC> ipconfig /all`
7. **Expected root cause:** Switchport Fa0/2 is assigned to VLAN 20 instead of VLAN 10.
8. **Steps to fix:** On S1: interface fa0/2; switchport mode access; switchport access vlan 10. Renew/reconfigure the PC address if needed.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Fa0/2 appears in VLAN 20; PC is expected in VLAN 10.` is no longer the observed failure evidence.

## VLAN-02 — Required user VLAN is missing from the switch

1. **Devices required:** Two PC-PT devices on 2960 ports Fa0/3 and Fa0/4; intended VLAN 30 and gateway 192.168.30.1.
2. **Basic topology:** Two PC-PT devices on 2960 ports Fa0/3 and Fa0/4; intended VLAN 30 and gateway 192.168.30.1.
3. **Initial working configuration:** Create the referenced VLAN(s) on each required 2960, configure PC-facing ports as access ports in their intended VLAN, and use a trunk for any switch-to-switch link. Configure the stated gateway if routing is involved.
4. **Fault to intentionally introduce:** VLAN 30 was deleted or never created on the switch.
5. **Expected symptom:** Engineering PCs on Fa0/3-Fa0/4 are isolated even though their ports reference VLAN 30.
6. **Commands to collect evidence:** `S1# show vlan brief; S1# show running-config interface fa0/3`
7. **Expected root cause:** VLAN 30 was deleted or never created on the switch.
8. **Steps to fix:** Create it with vlan 30; name ENGINEERING; then confirm both access ports are assigned to VLAN 30.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `show vlan brief has no VLAN 30; Fa0/3 configuration contains switchport access vlan 30.` is no longer the observed failure evidence.

## VLAN-03 — Inter-switch link is configured as an access port

1. **Devices required:** S1 Fa0/24 connected to S2 Fa0/24; VLAN 10 has a PC on each switch.
2. **Basic topology:** S1 Fa0/24 connected to S2 Fa0/24; VLAN 10 has a PC on each switch.
3. **Initial working configuration:** Create the referenced VLAN(s) on each required 2960, configure PC-facing ports as access ports in their intended VLAN, and use a trunk for any switch-to-switch link. Configure the stated gateway if routing is involved.
4. **Fault to intentionally introduce:** The inter-switch uplink is access mode rather than an 802.1Q trunk.
5. **Expected symptom:** VLAN 10 hosts on different 2960 switches cannot communicate, while same-switch hosts work.
6. **Commands to collect evidence:** `S1# show interfaces trunk; S1# show running-config interface fa0/24; S2# show interfaces trunk`
7. **Expected root cause:** The inter-switch uplink is access mode rather than an 802.1Q trunk.
8. **Steps to fix:** Configure both uplink ports: switchport mode trunk; optionally set switchport trunk native vlan 99 consistently. Verify with show interfaces trunk.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `No operational trunks are listed; Fa0/24 is configured with switchport mode access.` is no longer the observed failure evidence.

## VLAN-04 — VLAN is excluded from the allowed trunk list

1. **Devices required:** S1 and S2 trunk on Fa0/24; VLAN 40 PC on S1 and VLAN 40 server on S2.
2. **Basic topology:** S1 and S2 trunk on Fa0/24; VLAN 40 PC on S1 and VLAN 40 server on S2.
3. **Initial working configuration:** Create the referenced VLAN(s) on each required 2960, configure PC-facing ports as access ports in their intended VLAN, and use a trunk for any switch-to-switch link. Configure the stated gateway if routing is involved.
4. **Fault to intentionally introduce:** VLAN 40 is not allowed across the trunk.
5. **Expected symptom:** A VLAN 40 PC can reach its local gateway but not a VLAN 40 server across the switch uplink.
6. **Commands to collect evidence:** `S1# show interfaces trunk; S2# show interfaces trunk`
7. **Expected root cause:** VLAN 40 is not allowed across the trunk.
8. **Steps to fix:** On the relevant trunk: switchport trunk allowed vlan add 40. Confirm VLAN 40 is allowed and forwarding at both ends.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Fa0/24 is trunking, but VLANs allowed/active show 10,20,30 and omit 40.` is no longer the observed failure evidence.

## IP-01 — Host has an address from the wrong subnet

1. **Devices required:** PC-PT to 2960 to R1 G0/0; LAN gateway is 192.168.10.1/24.
2. **Basic topology:** PC-PT to 2960 to R1 G0/0; LAN gateway is 192.168.10.1/24.
3. **Initial working configuration:** Connect the devices described below with copper Ethernet (or wireless association), assign the addresses stated in the topology, enable router interfaces with `no shutdown`, and configure the referenced VLANs/routes/services before introducing the fault.
4. **Fault to intentionally introduce:** Static host IP belongs to a different subnet.
5. **Expected symptom:** PC cannot ping its default gateway and is assigned 192.168.20.50 while connected to the 192.168.10.0/24 LAN.
6. **Commands to collect evidence:** `PC> ipconfig /all; PC> ping 192.168.10.1; R1# show ip interface brief`
7. **Expected root cause:** Static host IP belongs to a different subnet.
8. **Steps to fix:** Set the PC to an unused 192.168.10.x address with mask 255.255.255.0, or use the correct DHCP pool.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `PC IPv4 address is 192.168.20.50/24; R1 G0/0 is 192.168.10.1 up/up.` is no longer the observed failure evidence.

## IP-02 — Host subnet mask is too broad

1. **Devices required:** PC-PT on 192.168.10.0/24 LAN; R1 routes to 192.168.11.0/24.
2. **Basic topology:** PC-PT on 192.168.10.0/24 LAN; R1 routes to 192.168.11.0/24.
3. **Initial working configuration:** Connect the devices described below with copper Ethernet (or wireless association), assign the addresses stated in the topology, enable router interfaces with `no shutdown`, and configure the referenced VLANs/routes/services before introducing the fault.
4. **Fault to intentionally introduce:** Host subnet mask is /16 instead of the LAN /24.
5. **Expected symptom:** PC reaches local addresses but sends traffic for 192.168.11.0/24 directly instead of through the gateway.
6. **Commands to collect evidence:** `PC> ipconfig /all; PC> tracert 192.168.11.10; R1# show ip route`
7. **Expected root cause:** Host subnet mask is /16 instead of the LAN /24.
8. **Steps to fix:** Configure mask 255.255.255.0 on the PC and retain gateway 192.168.10.1; retry the trace.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `PC mask is 255.255.0.0; tracert does not use 192.168.10.1 as first hop.` is no longer the observed failure evidence.

## IP-03 — Host has an incorrect default gateway

1. **Devices required:** PC-PT and R1 G0/0 on 192.168.10.0/24; remote server on 192.168.20.0/24.
2. **Basic topology:** PC-PT and R1 G0/0 on 192.168.10.0/24; remote server on 192.168.20.0/24.
3. **Initial working configuration:** Connect the devices described below with copper Ethernet (or wireless association), assign the addresses stated in the topology, enable router interfaces with `no shutdown`, and configure the referenced VLANs/routes/services before introducing the fault.
4. **Fault to intentionally introduce:** The PC default gateway is not the router interface on its subnet.
5. **Expected symptom:** PC can ping same-subnet peers but cannot reach a remote server.
6. **Commands to collect evidence:** `PC> ipconfig /all; PC> ping 192.168.10.1; PC> tracert 192.168.20.10`
7. **Expected root cause:** The PC default gateway is not the router interface on its subnet.
8. **Steps to fix:** Set the PC default gateway to 192.168.10.1 and verify a remote ping.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `PC gateway is 192.168.10.254, while R1 interface/gateway is 192.168.10.1.` is no longer the observed failure evidence.

## IP-04 — Duplicate static IP address

1. **Devices required:** Two PC-PT devices on one 2960 access VLAN 10, with R1 gateway 192.168.10.1.
2. **Basic topology:** Two PC-PT devices on one 2960 access VLAN 10, with R1 gateway 192.168.10.1.
3. **Initial working configuration:** Connect the devices described below with copper Ethernet (or wireless association), assign the addresses stated in the topology, enable router interfaces with `no shutdown`, and configure the referenced VLANs/routes/services before introducing the fault.
4. **Fault to intentionally introduce:** Two endpoints use the same static IPv4 address.
5. **Expected symptom:** Two PCs intermittently lose connectivity and ARP resolves the same IP to changing MAC addresses.
6. **Commands to collect evidence:** `PC-A> ipconfig /all; PC-B> ipconfig /all; PC-A> ping 192.168.10.60; PC-A> arp -a`
7. **Expected root cause:** Two endpoints use the same static IPv4 address.
8. **Steps to fix:** Assign one PC a unique unused address (or DHCP), clear/refresh ARP, and confirm unique addressing.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Both PCs are configured as 192.168.10.60/24; the ARP MAC for .60 changes after traffic.` is no longer the observed failure evidence.

## DHCP-01 — DHCP service is disabled on the server

1. **Devices required:** PC-PT to 2960 to Server-PT; server DHCP service supplies 192.168.50.0/24.
2. **Basic topology:** PC-PT to 2960 to Server-PT; server DHCP service supplies 192.168.50.0/24.
3. **Initial working configuration:** Configure the stated LAN gateway and an enabled DHCP service/pool whose network, default gateway, DNS option, and usable range match the topology. Confirm one client gets a lease.
4. **Fault to intentionally introduce:** The DHCP service is disabled.
5. **Expected symptom:** New clients remain at 0.0.0.0 or APIPA-style addressing and cannot join the LAN.
6. **Commands to collect evidence:** `PC> ipconfig /all; Server-PT Services > DHCP; R1# show ip dhcp binding`
7. **Expected root cause:** The DHCP service is disabled.
8. **Steps to fix:** Enable DHCP on Server-PT Services > DHCP, ensure the pool is enabled, then renew the PC lease.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `PC has no valid lease; the Server-PT DHCP service toggle is Off; router has no bindings.` is no longer the observed failure evidence.

## DHCP-02 — DHCP pool advertises the wrong default gateway

1. **Devices required:** Server-PT DHCP server on 192.168.60.0/24; R1 G0/0 is 192.168.60.1.
2. **Basic topology:** Server-PT DHCP server on 192.168.60.0/24; R1 G0/0 is 192.168.60.1.
3. **Initial working configuration:** Configure the stated LAN gateway and an enabled DHCP service/pool whose network, default gateway, DNS option, and usable range match the topology. Confirm one client gets a lease.
4. **Fault to intentionally introduce:** DHCP pool default-router option is incorrect.
5. **Expected symptom:** Client receives a valid IP lease but cannot reach any remote subnet.
6. **Commands to collect evidence:** `PC> ipconfig /all; Server-PT Services > DHCP; PC> ping 192.168.60.1`
7. **Expected root cause:** DHCP pool default-router option is incorrect.
8. **Steps to fix:** Change the pool default gateway/default-router to 192.168.60.1, save, then release and renew the client lease.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `PC lease is 192.168.60.50/24 but gateway option is 192.168.60.254; the router is .1.` is no longer the observed failure evidence.

## DHCP-03 — DHCP pool has no available addresses

1. **Devices required:** Server-PT DHCP pool 192.168.70.0/24 deliberately limited to two assignable addresses; three PCs.
2. **Basic topology:** Server-PT DHCP pool 192.168.70.0/24 deliberately limited to two assignable addresses; three PCs.
3. **Initial working configuration:** Configure the stated LAN gateway and an enabled DHCP service/pool whose network, default gateway, DNS option, and usable range match the topology. Confirm one client gets a lease.
4. **Fault to intentionally introduce:** The DHCP scope is exhausted.
5. **Expected symptom:** Additional clients fail to obtain addresses while earlier clients retain working leases.
6. **Commands to collect evidence:** `Server-PT Services > DHCP; PC-3> ipconfig /all; R1# show ip dhcp binding`
7. **Expected root cause:** The DHCP scope is exhausted.
8. **Steps to fix:** Increase the pool range or reduce lease use, avoiding reserved gateway/static addresses; renew the affected client.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `All configured pool addresses are leased; PC-3 has no valid lease.` is no longer the observed failure evidence.

## DHCP-04 — Router DHCP pool has the wrong network statement

1. **Devices required:** R1 G0/0 192.168.80.1/24 to 2960 and PC-PT; R1 provides DHCP.
2. **Basic topology:** R1 G0/0 192.168.80.1/24 to 2960 and PC-PT; R1 provides DHCP.
3. **Initial working configuration:** Configure the stated LAN gateway and an enabled DHCP service/pool whose network, default gateway, DNS option, and usable range match the topology. Confirm one client gets a lease.
4. **Fault to intentionally introduce:** Router DHCP pool network does not match the client LAN.
5. **Expected symptom:** Client receives no lease from a router acting as DHCP server despite an up LAN interface.
6. **Commands to collect evidence:** `R1# show running-config | section dhcp; R1# show ip dhcp pool; PC> ipconfig /all`
7. **Expected root cause:** Router DHCP pool network does not match the client LAN.
8. **Steps to fix:** Correct the pool network to 192.168.80.0 255.255.255.0, exclude infrastructure IPs, and renew the client.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Pool network is 192.168.81.0 255.255.255.0 while G0/0 is 192.168.80.1/24; no binding appears.` is no longer the observed failure evidence.

## DNS-01 — DHCP supplies an unreachable DNS server

1. **Devices required:** PC receives DHCP on 192.168.90.0/24; DNS Server-PT is 192.168.90.10.
2. **Basic topology:** PC receives DHCP on 192.168.90.0/24; DNS Server-PT is 192.168.90.10.
3. **Initial working configuration:** Configure client addressing and reachability to Server-PT. Enable DNS and create the required record unless the fault specifically disables/removes it.
4. **Fault to intentionally introduce:** DNS server option on the host/DHCP pool is wrong.
5. **Expected symptom:** Client can ping the web server IP but cannot browse or ping it by hostname.
6. **Commands to collect evidence:** `PC> ipconfig /all; PC> ping 192.168.90.10; PC> nslookup intranet.local`
7. **Expected root cause:** DNS server option on the host/DHCP pool is wrong.
8. **Steps to fix:** Set the DHCP DNS server option (or static client DNS) to 192.168.90.10, renew the lease, and retest the name.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `PC DNS server is 192.168.90.99; .99 does not respond, though .10 is the DNS server.` is no longer the observed failure evidence.

## DNS-02 — DNS A record for internal site is missing

1. **Devices required:** PC and DNS/Web Server-PT on one routed network; intended A record intranet.local -> 10.10.10.10.
2. **Basic topology:** PC and DNS/Web Server-PT on one routed network; intended A record intranet.local -> 10.10.10.10.
3. **Initial working configuration:** Configure client addressing and reachability to Server-PT. Enable DNS and create the required record unless the fault specifically disables/removes it.
4. **Fault to intentionally introduce:** Required DNS A record is absent.
5. **Expected symptom:** The intranet server is reachable by IP address but intranet.local cannot be resolved.
6. **Commands to collect evidence:** `PC> nslookup intranet.local; PC> ping 10.10.10.10; Server-PT Services > DNS`
7. **Expected root cause:** Required DNS A record is absent.
8. **Steps to fix:** Add A record intranet.local with address 10.10.10.10 on Server-PT, then query the name again.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `nslookup reports no record for intranet.local; ping to 10.10.10.10 succeeds; DNS service is enabled.` is no longer the observed failure evidence.

## DNS-03 — DNS service is disabled

1. **Devices required:** PC-PT and Server-PT on 10.10.20.0/24; Server-PT hosts DNS records.
2. **Basic topology:** PC-PT and Server-PT on 10.10.20.0/24; Server-PT hosts DNS records.
3. **Initial working configuration:** Configure client addressing and reachability to Server-PT. Enable DNS and create the required record unless the fault specifically disables/removes it.
4. **Fault to intentionally introduce:** DNS service is disabled on the server.
5. **Expected symptom:** All name lookups fail, although clients can reach the DNS server by IP.
6. **Commands to collect evidence:** `PC> ping 10.10.20.10; PC> nslookup portal.local; Server-PT Services > DNS`
7. **Expected root cause:** DNS service is disabled on the server.
8. **Steps to fix:** Enable DNS on Server-PT Services > DNS and verify existing records before retrying nslookup.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Ping to 10.10.20.10 succeeds; DNS service toggle is Off; queries time out/fail.` is no longer the observed failure evidence.

## ROUTE-01 — Static route to remote LAN is missing

1. **Devices required:** R1 192.168.1.0/24 -- 10.0.12.0/30 -- R2 192.168.2.0/24.
2. **Basic topology:** R1 192.168.1.0/24 -- 10.0.12.0/30 -- R2 192.168.2.0/24.
3. **Initial working configuration:** Address each router interface as stated, issue `no shutdown`, and install correct reciprocal routes (or matching OSPF area 0 statements) so end-to-end pings work before the fault.
4. **Fault to intentionally introduce:** R1 lacks a route to the R2 LAN.
5. **Expected symptom:** Hosts on R1 LAN cannot reach hosts behind R2, despite an up serial/Ethernet transit link.
6. **Commands to collect evidence:** `R1# show ip route; R1# show ip interface brief; PC1> tracert 192.168.2.10`
7. **Expected root cause:** R1 lacks a route to the R2 LAN.
8. **Steps to fix:** Add ip route 192.168.2.0 255.255.255.0 10.0.12.2 on R1 and ensure R2 has a return route.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `R1 route table contains connected networks only; no route to 192.168.2.0/24; transit interfaces are up/up.` is no longer the observed failure evidence.

## ROUTE-02 — Static route points to an invalid next hop

1. **Devices required:** R1 and R2 connected on 10.0.12.0/30; R2 serves 172.16.2.0/24.
2. **Basic topology:** R1 and R2 connected on 10.0.12.0/30; R2 serves 172.16.2.0/24.
3. **Initial working configuration:** Address each router interface as stated, issue `no shutdown`, and install correct reciprocal routes (or matching OSPF area 0 statements) so end-to-end pings work before the fault.
4. **Fault to intentionally introduce:** Static route uses the wrong next-hop address.
5. **Expected symptom:** Traffic to a remote network fails at R1 even though a static route is present.
6. **Commands to collect evidence:** `R1# show ip route 172.16.2.0; R1# show running-config | include ^ip route; PC> tracert 172.16.2.10`
7. **Expected root cause:** Static route uses the wrong next-hop address.
8. **Steps to fix:** Remove the bad route and add ip route 172.16.2.0 255.255.255.0 10.0.12.2; verify return routing.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Route reads S 172.16.2.0/24 via 10.0.12.6; .6 is not R2 on the transit subnet.` is no longer the observed failure evidence.

## ROUTE-03 — Edge router has no default route

1. **Devices required:** R1 LAN 192.168.100.0/24 to ISP router over 203.0.113.0/30; server beyond ISP.
2. **Basic topology:** R1 LAN 192.168.100.0/24 to ISP router over 203.0.113.0/30; server beyond ISP.
3. **Initial working configuration:** Address each router interface as stated, issue `no shutdown`, and install correct reciprocal routes (or matching OSPF area 0 statements) so end-to-end pings work before the fault.
4. **Fault to intentionally introduce:** Default route toward the ISP is missing.
5. **Expected symptom:** Internal clients reach internal networks but cannot reach an ISP test server.
6. **Commands to collect evidence:** `R1# show ip route; R1# show running-config | include ^ip route; PC> tracert 198.51.100.10`
7. **Expected root cause:** Default route toward the ISP is missing.
8. **Steps to fix:** Configure ip route 0.0.0.0 0.0.0.0 203.0.113.2 on R1 and validate ISP return routing/NAT as applicable.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `R1 has connected LAN and WAN routes but no Gateway of last resort/default route.` is no longer the observed failure evidence.

## ROUTE-04 — Router LAN interface is administratively down

1. **Devices required:** PC-PT through 2960 to R1 G0/1, addressed 192.168.110.1/24.
2. **Basic topology:** PC-PT through 2960 to R1 G0/1, addressed 192.168.110.1/24.
3. **Initial working configuration:** Address each router interface as stated, issue `no shutdown`, and install correct reciprocal routes (or matching OSPF area 0 statements) so end-to-end pings work before the fault.
4. **Fault to intentionally introduce:** The LAN router interface has been shut down.
5. **Expected symptom:** Every host on one LAN loses its gateway and remote connectivity after an interface change.
6. **Commands to collect evidence:** `R1# show ip interface brief; R1# show running-config interface g0/1; PC> ping 192.168.110.1`
7. **Expected root cause:** The LAN router interface has been shut down.
8. **Steps to fix:** Enter interface g0/1 and issue no shutdown; verify status up/up and test gateway ping.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `G0/1 shows 192.168.110.1 with Status administratively down and Protocol down.` is no longer the observed failure evidence.

## ROUTE-05 — OSPF advertises the wrong LAN network

1. **Devices required:** R1 and R2 run OSPF area 0 over 10.1.12.0/30; R1 LAN is 10.1.1.0/24.
2. **Basic topology:** R1 and R2 run OSPF area 0 over 10.1.12.0/30; R1 LAN is 10.1.1.0/24.
3. **Initial working configuration:** Address each router interface as stated, issue `no shutdown`, and install correct reciprocal routes (or matching OSPF area 0 statements) so end-to-end pings work before the fault.
4. **Fault to intentionally introduce:** R1 OSPF configuration does not include its LAN interface network.
5. **Expected symptom:** R2 does not learn R1 LAN route, while the inter-router adjacency is up.
6. **Commands to collect evidence:** `R1# show running-config | section router ospf; R2# show ip route ospf; R1# show ip ospf neighbor`
7. **Expected root cause:** R1 OSPF configuration does not include its LAN interface network.
8. **Steps to fix:** On R1 add network 10.1.1.0 0.0.0.255 area 0 (or enable OSPF on G0/0); confirm route appears on R2.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Neighbor is FULL; R1 OSPF network statement matches 10.1.12.0 but not 10.1.1.0; R2 lacks 10.1.1.0/24.` is no longer the observed failure evidence.

## ACL-01 — Inbound ACL blocks approved HTTPS traffic

1. **Devices required:** Client LAN to R1 to Server-PT 10.20.20.10; extended ACL filters client VLAN inbound.
2. **Basic topology:** Client LAN to R1 to Server-PT 10.20.20.10; extended ACL filters client VLAN inbound.
3. **Initial working configuration:** Configure end-to-end routing first. Create an ACL that permits the intended baseline traffic and bind it only where policy requires; verify the permitted flow.
4. **Fault to intentionally introduce:** ACL explicitly denies legitimate HTTPS traffic.
5. **Expected symptom:** Users cannot open the internal HTTPS server, while other allowed services work.
6. **Commands to collect evidence:** `R1# show access-lists; R1# show running-config interface g0/0; PC> ping 10.20.20.10`
7. **Expected root cause:** ACL explicitly denies legitimate HTTPS traffic.
8. **Steps to fix:** Replace/remove the deny and add an appropriate permit tcp rule for HTTPS before any general deny; retest browser access.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `ACL contains deny tcp 10.20.10.0 0.0.0.255 host 10.20.20.10 eq 443 with match counter increasing.` is no longer the observed failure evidence.

## ACL-02 — ACL is applied to the wrong interface

1. **Devices required:** R1 G0/0 is management LAN; G0/1 is WAN; ACL intended for WAN traffic.
2. **Basic topology:** R1 G0/0 is management LAN; G0/1 is WAN; ACL intended for WAN traffic.
3. **Initial working configuration:** Configure end-to-end routing first. Create an ACL that permits the intended baseline traffic and bind it only where policy requires; verify the permitted flow.
4. **Fault to intentionally introduce:** The ACL is bound inbound on the management LAN interface rather than the intended WAN interface.
5. **Expected symptom:** Management workstation cannot reach devices on its local VLAN after an Internet-filter ACL was added.
6. **Commands to collect evidence:** `R1# show running-config interface g0/0; R1# show running-config interface g0/1; R1# show access-lists`
7. **Expected root cause:** The ACL is bound inbound on the management LAN interface rather than the intended WAN interface.
8. **Steps to fix:** Remove ip access-group INTERNET-FILTER in from G0/0 and apply it to the correct interface/direction after policy review.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `G0/0 has ip access-group INTERNET-FILTER in; G0/1 has none; ACL denies management source traffic.` is no longer the observed failure evidence.

## ACL-03 — ACL direction is reversed

1. **Devices required:** Client LAN on R1 G0/0, server LAN on G0/1; ACL should filter client traffic near source.
2. **Basic topology:** Client LAN on R1 G0/0, server LAN on G0/1; ACL should filter client traffic near source.
3. **Initial working configuration:** Configure end-to-end routing first. Create an ACL that permits the intended baseline traffic and bind it only where policy requires; verify the permitted flow.
4. **Fault to intentionally introduce:** ACL is applied in the wrong direction/interface path.
5. **Expected symptom:** Traffic initiated by clients is unexpectedly blocked although the ACL entries match the intended client subnet.
6. **Commands to collect evidence:** `R1# show running-config interface g0/0; R1# show running-config interface g0/1; R1# show access-lists`
7. **Expected root cause:** ACL is applied in the wrong direction/interface path.
8. **Steps to fix:** Remove the misplaced access group and apply it inbound on G0/0 (or outbound on G0/1 according to policy); confirm counters increment correctly.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `ACL is applied outbound on G0/0; match counters remain zero while client-to-server packets exit G0/1.` is no longer the observed failure evidence.

## NAT-01 — PAT is not configured for internal users

1. **Devices required:** R1 inside LAN 192.168.120.0/24 and outside link 203.0.113.2/30 to ISP server.
2. **Basic topology:** R1 inside LAN 192.168.120.0/24 and outside link 203.0.113.2/30 to ISP server.
3. **Initial working configuration:** Configure inside LAN, outside ISP link, default/return routing, `ip nat inside`/`ip nat outside`, and an ACL/PAT rule that translates the stated user subnet(s). Verify an external ping.
4. **Fault to intentionally introduce:** NAT/PAT translation rule is missing.
5. **Expected symptom:** Internal PCs can reach the edge router but cannot reach an external server that lacks routes back to private space.
6. **Commands to collect evidence:** `R1# show ip nat translations; R1# show running-config | section ip nat; PC> tracert 198.51.100.10`
7. **Expected root cause:** NAT/PAT translation rule is missing.
8. **Steps to fix:** Mark interfaces inside/outside and configure ACL plus ip nat inside source list <acl> interface g0/1 overload; generate traffic and inspect translations.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `No NAT translations appear after traffic; configuration has no ip nat inside source statement.` is no longer the observed failure evidence.

## NAT-02 — NAT inside and outside roles are reversed

1. **Devices required:** R1 G0/0 faces 192.168.130.0/24; G0/1 faces ISP 203.0.113.2/30; PAT rule exists.
2. **Basic topology:** R1 G0/0 faces 192.168.130.0/24; G0/1 faces ISP 203.0.113.2/30; PAT rule exists.
3. **Initial working configuration:** Configure inside LAN, outside ISP link, default/return routing, `ip nat inside`/`ip nat outside`, and an ACL/PAT rule that translates the stated user subnet(s). Verify an external ping.
4. **Fault to intentionally introduce:** NAT inside/outside interface designations are reversed.
5. **Expected symptom:** NAT translation table stays empty and Internet access fails after interface role changes.
6. **Commands to collect evidence:** `R1# show running-config interface g0/0; R1# show running-config interface g0/1; R1# show ip nat translations`
7. **Expected root cause:** NAT inside/outside interface designations are reversed.
8. **Steps to fix:** On G0/0 use ip nat inside; on G0/1 use ip nat outside; clear translations if needed and test external connectivity.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `G0/0 is marked ip nat outside and G0/1 is marked ip nat inside, opposite the topology.` is no longer the observed failure evidence.

## NAT-03 — NAT ACL omits the user subnet

1. **Devices required:** R1 router-on-a-stick: VLAN 130 and VLAN 140; PAT ACL should cover both; one ISP link.
2. **Basic topology:** R1 router-on-a-stick: VLAN 130 and VLAN 140; PAT ACL should cover both; one ISP link.
3. **Initial working configuration:** Configure inside LAN, outside ISP link, default/return routing, `ip nat inside`/`ip nat outside`, and an ACL/PAT rule that translates the stated user subnet(s). Verify an external ping.
4. **Fault to intentionally introduce:** NAT matching ACL/rule excludes 192.168.140.0/24.
5. **Expected symptom:** One internal VLAN has Internet access but users in VLAN 140 do not.
6. **Commands to collect evidence:** `R1# show access-lists; R1# show ip nat translations; PC-V140> ping 198.51.100.10`
7. **Expected root cause:** NAT matching ACL/rule excludes 192.168.140.0/24.
8. **Steps to fix:** Add permit 192.168.140.0 0.0.0.255 to the NAT ACL (without breaking existing entries); create traffic and confirm a translation.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `NAT ACL permits 192.168.130.0/24 only; VLAN 140 packets create no translations.` is no longer the observed failure evidence.

## WLAN-01 — Laptop joins the wrong SSID

1. **Devices required:** Wireless Router/AP broadcasts StaffNet and GuestNet; Laptop-PT should use StaffNet.
2. **Basic topology:** Wireless Router/AP broadcasts StaffNet and GuestNet; Laptop-PT should use StaffNet.
3. **Initial working configuration:** Configure the AP/router LAN addressing, intended SSID and WPA2 settings, and the stated VLAN or server reachability. Associate the laptop to the intended SSID and verify a working address.
4. **Fault to intentionally introduce:** Client selected the wrong SSID.
5. **Expected symptom:** Laptop associates to a nearby guest SSID and cannot access required staff resources.
6. **Commands to collect evidence:** `Laptop-PT Desktop > PC Wireless; Wireless Router GUI > Wireless; Laptop> ipconfig /all`
7. **Expected root cause:** Client selected the wrong SSID.
8. **Steps to fix:** Select StaffNet in Laptop-PT PC Wireless, use its matching credentials, reconnect, and verify the staff subnet address.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Laptop wireless profile/association shows GuestNet; StaffNet is the required corporate SSID.` is no longer the observed failure evidence.

## WLAN-02 — Wireless security key does not match AP

1. **Devices required:** Laptop-PT connects to Wireless Router/AP StaffNet using WPA2-PSK.
2. **Basic topology:** Laptop-PT connects to Wireless Router/AP StaffNet using WPA2-PSK.
3. **Initial working configuration:** Configure the AP/router LAN addressing, intended SSID and WPA2 settings, and the stated VLAN or server reachability. Associate the laptop to the intended SSID and verify a working address.
4. **Fault to intentionally introduce:** Wireless pre-shared key is incorrect.
5. **Expected symptom:** Laptop sees StaffNet but authentication/association fails and no IP address is assigned.
6. **Commands to collect evidence:** `Laptop-PT Desktop > PC Wireless; Wireless Router GUI > Wireless Security; Laptop> ipconfig /all`
7. **Expected root cause:** Wireless pre-shared key is incorrect.
8. **Steps to fix:** Enter the exact WPA2-PSK configured on the AP into the laptop profile, reconnect, then obtain an address.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `AP uses WPA2-PSK key Cisco12345; laptop profile has a different key and remains disconnected.` is no longer the observed failure evidence.

## WLAN-03 — Wireless client uses static IP from wrong subnet

1. **Devices required:** Wireless Router/AP LAN is 192.168.150.0/24 with gateway 192.168.150.1; Laptop-PT joins StaffNet.
2. **Basic topology:** Wireless Router/AP LAN is 192.168.150.0/24 with gateway 192.168.150.1; Laptop-PT joins StaffNet.
3. **Initial working configuration:** Configure the AP/router LAN addressing, intended SSID and WPA2 settings, and the stated VLAN or server reachability. Associate the laptop to the intended SSID and verify a working address.
4. **Fault to intentionally introduce:** Wireless client IPv4 configuration belongs to the wrong subnet.
5. **Expected symptom:** Laptop associates successfully but cannot reach the AP gateway or LAN servers.
6. **Commands to collect evidence:** `Laptop> ipconfig /all; Laptop> ping 192.168.150.1; Wireless Router GUI > Setup`
7. **Expected root cause:** Wireless client IPv4 configuration belongs to the wrong subnet.
8. **Steps to fix:** Set laptop IPv4 to DHCP or a unique 192.168.150.x/24 address with gateway 192.168.150.1; verify gateway ping.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Laptop is associated but has static 192.168.151.50/24 and gateway 192.168.151.1; AP LAN is 192.168.150.1/24.` is no longer the observed failure evidence.

## WLAN-04 — Guest Wi-Fi can reach an internal server

1. **Devices required:** Wireless router/AP provides Guest VLAN 160 and Staff/Server VLAN 170; R1 inter-VLAN routing applies policy ACL.
2. **Basic topology:** Wireless router/AP provides Guest VLAN 160 and Staff/Server VLAN 170; R1 inter-VLAN routing applies policy ACL.
3. **Initial working configuration:** Configure the AP/router LAN addressing, intended SSID and WPA2 settings, and the stated VLAN or server reachability. Associate the laptop to the intended SSID and verify a working address.
4. **Fault to intentionally introduce:** Guest wireless VLAN lacks an isolation ACL for the internal server network.
5. **Expected symptom:** Guest laptop successfully pings a protected internal server, violating segmentation policy.
6. **Commands to collect evidence:** `Guest Laptop> ping 192.168.170.10; R1# show access-lists; R1# show running-config interface g0/0.160`
7. **Expected root cause:** Guest wireless VLAN lacks an isolation ACL for the internal server network.
8. **Steps to fix:** Apply an extended ACL inbound on the guest VLAN interface to deny guest-to-internal traffic while permitting required Internet/DHCP/DNS traffic; verify server ping fails and Internet still works.
9. **Verification command:** Run the originally failed `ping`, `tracert`, hostname lookup, or service test again; then confirm: `Guest ping to 192.168.170.10 succeeds; guest VLAN interface has no ACL blocking 192.168.170.0/24.` is no longer the observed failure evidence.
