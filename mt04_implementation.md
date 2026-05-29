The following specific objectives will be implemented:

### Restructuring of the Network Topology
**Segment the network into three distinct logical zones** to guarantee security and performance:
* **LAN/Departmental Zone:** Subnet `192.168.1.0/24` (Internet Connection).
* **DMZ Zone:** Subnet `10.0.0.0/8` (Public Servers).
* **LAN/Student Zone:** Subnet `172.16.0.0/16` (Internal classroom network).

**Configure inter-VLAN routing** to manage traffic between the different workgroups (1SMR, 2SMR, 1ASIR, 2ASIR, etc.):
* With OPNsense.

### Implementation of Services in the DMZ
* **DNS:** Configuration of name resolution and web content filtering, optimizing bandwidth through caching.
* **Web:** Hosting of the final ASIR project.
* **Database:** Setup of a MySQL Server for linkage with the web server.
* **OPNsense:** Routing and network management service.
* **LDAP:** Authentication server for Proxmox and WordPress.

### Optimization of the Internal Network (Management Services)
* **FOG Project:** Implementation in the distribution network for cloning and deploying operating system images over the network (PXE/Multicast), eliminating the use of physical media.

### External Connectivity
* **Tailscale:** Secure connection to our server via the Internet.

## 1.3. Project Scope
The project covers:

### Complete lifecycle of the solution at a logical and software level:
* **Design:** Topology modeling in Cisco Packet Tracer, defining IP addressing, VLANs, and static/dynamic routing.
* **Implementation:** Installation and configuration of services (DNS, Web, Database, FOG) on server operating systems (Linux/Windows Server).
* **Security:** Definition of firewall rules and ACLs to control traffic between the Student network, the DMZ, and the Internet.
* **Validation:** Execution of a test plan for connectivity and service functionality.

### Hardware analysis, maintenance, and configuration:

#### Inventory and Compatibility Audit:
* Evaluation of existing hardware resources (CPU, RAM, Storage) on servers intended for virtualization (Proxmox/ESXi) to ensure they support the load of the new services (FOG, Web, etc.).
* Verification of capabilities in interconnection equipment: Support for standards in switches and processing capacity/throughput in the router to manage DMZ traffic without bottlenecks.

#### Preventive Maintenance and Updating:
* Firmware/IOS updates on switches and routers to mitigate known vulnerabilities and ensure compatibility with new features.
* Configuration of RAID levels (software or hardware) on servers to guarantee data redundancy and availability.

#### Active Device Configuration:
* Configuration of interfaces, static/dynamic routing, and base services (DHCP, NTP) on network hardware.
* Port segmentation on switches (Access/Trunk) according to the proposed VLAN design.

#### Passive Physical Infrastructure:
* Installation/inspection of structured cabling (UTP/Fiber), trunking, wall plates (rosettes), and patch panels. The already certified cabling of the building will be used.
* Includes the physical assembly of rack cabinets and the associated electrical installation (UPS).
