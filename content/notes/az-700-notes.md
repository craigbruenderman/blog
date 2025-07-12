---
title: Azure AZ-700 Notes
author: Craig Bruenderman
geometry: margin=2cm
toc: true
output: pdf_document
---

## Topics

* Private Endpoint
* vNET Service Endpoints
* Application Security Groups within NSGs
* Azure NAT Gateway max 16 public IPs
* NAT Gateway only supports TCP and UDP (no ICMP)

## Notes

* WAF needs dedicated subnet
* Virtual Appliance next hop type for virtual FW

## Azure Extended Network

* Windows 2019 Server on-prem and in Azure acting as VTEPs for L2 extension
* MS wants you to do this as a nested VM running under Win2022 server for some reason

## VPN

### Virtual Network Gateways

* In order to move from Basic SKU to VPNGW SKU, you have to delete and recreate
* VPNGW SKU can be resized within the same generation

### Site to Site

### Point to Site

* Auth type options are certificate, RADIUS, and Azure AD
* Authenticate using native Azure certificate authentication - When using the native Azure certificate authentication, a client certificate that is present on the device is used to authenticate the connecting user.
* Authenticate using native Azure Active Directory authentication - Azure AD authentication allows users to connect to Azure using their Azure Active Directory credentials. Native Azure AD authentication is only supported for OpenVPN protocol and Windows 10 and requires the use of the Azure VPN Client
* Authenticate using Active Directory (AD) Domain Server - AD Domain authentication allows users to connect to Azure using their organization domain credentials. It requires a RADIUS server that integrates with the AD server. Organizations can also leverage their existing RADIUS deployment.
* Supported Tunnel Types
  * IKEv1 and IKEv2
  * OpenVPN
  * SSTP

## ExpressRoute

* Microsoft (Public) Peering
  * Reaching (typically public) MS Services over private line
* Azure Private Peering
* ExpressRoute Premium connects to all GeoPolitical regions
* GlobalReach 
* ExpressRoute Direct
  * Bypasses provider partner for a direct connection into an MS Edge location
  * Offers 100Gbps
* ExpressRoute Virtual Network Gateways can dual purpose as VPN Gateways if they are above Standard/ErGw1AZ SKU
  * Standard will advertise/learn 500/4000 routes
  * High Performance will advertise/learn 500/9500 routes
  * High Performance will advertise/learn 500/9500 routes
* Service Key uniquely identifies the ExpressRoute when you need to troubleshoot

### Fast Path

## vNETs

### Subnets

* Every vNET must have at least 1 subnet
* Azure takes 5 addresses of each subnet
  * Network, broadcast, gateway, DNS
* Subnet to Subnet comms are permitted by default
  * Can be restricted with subnet assigned NSG

### Cross vNET Connectivity

* vNET Peering within same region
* Global vNET Peering across Azure regions
* Can also use VPN Gateways to transit between vNETs

### Private Link

* Azure Private Link Azure Private Link enables you to access Azure PaaS Services (for example, Azure Storage and SQL Database) and Azure hosted customer-owned/partner services over a private endpoint in your virtual network.
* A Private Link is essentially a link to a Standard Internal Load Balancer that can be shared to another Azure environment

### Service Endpoints

* Bridge between PaaS side of Azure and services side of Azure
* You can select private access and delegate this to specific subnets within a vNET in same region as the service
* Traffic stays off public Internet

### Subnet Delegation

* Sort of a deeper version of Service Endpoints
* You delegate your vNNET address space to an Azure Managed Service for addressing its instance endpoints

## DNS

* DNS forwarding also enables DNS resolution between virtual networks, and allows your on-premises machines to resolve Azure-provided host names
  * In order to resolve a VM's host name, the DNS server VM must reside in the same virtual network, and be configured to forward host name queries to Azure

### Private DNS

Azure Private DNS provides a reliable and secure DNS service for your virtual network. Azure Private DNS manages and resolves domain names in the virtual network without the need to configure a custom DNS solution. By using private DNS zones, you can use your own custom domain name instead of the Azure-provided names during deployment. Using a custom domain name helps you tailor your virtual network architecture to best suit your organization's needs. It provides a naming resolution for virtual machines (VMs) within a virtual network and connected virtual networks. Additionally, you can configure zones names with a split-horizon view, which allows a private and a public DNS zone to share the name.

## Azure Firewall

* Priority settings can be any number between 100 and 65000. With 100 being the highest priority.
* Azure firewall needs dedicated subnet

### Forced Tunnelling

* If forced tunneling was enabled, the Firewall Subnet would be named AzureFirewallManagementSubnet
* Forced tunneling can only be enabled during the creation of the firewall. It cannot be enabled after the firewall has been deployed.

## Virtual WAN

* Basic supports S2S only
  * No P2P or ExpressRoute
* Standard supports
  * ExpressRoute
  * P2S
  * S2S
  * Azure Firewall
  * NVA
  * Inter-hub and vNET-vNET transit

### Scaling

* A scale unit is a unit defined to pick an aggregate throughput of a gateway in Virtual hub
* 1 scale unit of VPN = 500 Mbps
* 1 scale unit of ExpressRoute = 2 Gbps
  * Example: 10 scale unit of VPN would imply 500 Mbps * 10 = 5 Gbps

## Azure Load Balancer

* Regional service
* Basic typically for test workloads only
  * Probes
    * TCP
    * HTTP
* Standard supports
  * Zone-redundancy
  * Outbound rules via NAT configuration

## Azure Application Gateway

* Regional service
* Layer 7 HTTP/HTTPS application load balancer
* Supports end-to-end encryption of traffic by terminating SSL connection at the application gateway. The gateway then applies the routing rules to the traffic, re-encrypts the packet, and forwards the packet to the appropriate back-end server based on the routing rules defined.

## Azure Traffic Manager

### Routing Methods

* Priority: Select Priority routing when you want to have a primary service endpoint for all traffic. You can provide multiple backup endpoints in case the primary or one of the backup endpoints is unavailable.
* Weighted: Select Weighted routing when you want to distribute traffic across a set of endpoints based on their weight. Set the weight the same to distribute evenly across all endpoints.
* Performance: Select Performance routing when you have endpoints in different geographic locations and you want end users to use the "closest" endpoint for the lowest network latency.
* Geographic: Select Geographic routing to direct users to specific endpoints (Azure, External, or Nested) based on where their DNS queries originate from geographically. With this routing method, it enables you to be in compliance with scenarios such as data sovereignty mandates, localization of content & user experience and measuring traffic from different regions.
* Multivalue: Select MultiValue for Traffic Manager profiles that can only have IPv4/IPv6 addresses as endpoints. When a query is received for this profile, all healthy endpoints are returned.
* Subnet: Select Subnet traffic-routing method to map sets of end-user IP address ranges to a specific endpoint. When a request is received, the endpoint returned will be the one mapped for that request’s source IP address. 

## Virtual NAT Gateway

* Primarily used to provide outbound network connectivity to vNETs
* Let's you deterministically set the public IP used for outbound
* Each NAT Gateway supports up to 16 public IPs
* NAT Gateways get associated with one or more subnets
* Doesn't work with basic SKU Public IPs

## Monitoring

* Network Performance Monitor - Network Performance Monitor is a cloud-based hybrid network monitoring solution that helps you monitor network performance between various points in your network infrastructure. It also helps you monitor network connectivity to service and application endpoints and monitor the performance of Azure ExpressRoute. You can monitor network connectivity across cloud deployments and on-premises locations, multiple data centers, and branch offices and mission-critical multitier applications or microservices. With Performance Monitor, you can detect network issues before users complain.
* IP flow verify - The IP flow verify capability enables you to specify a source and destination IPv4 address, port, protocol (TCP or UDP), and traffic direction (inbound or outbound). IP flow verify then tests the communication and informs you if the connection succeeds or fails.
* Connection monitor - Connection Monitor provides you RTT values on a per-minute granularity. The connection monitor capability monitors communication at a regular interval and informs you of reachability, latency, and network topology changes between the VM and the endpoint.
* Connection Troubleshoot - The connection troubleshoot capability enables you to test a connection between a VM and another VM, an FQDN, a URI, or an IPv4 address. The test returns similar information returned when using the connection monitor capability, but tests the connection at a point in time, rather than monitoring it over time, as connection monitor does
* IP flow verify in Azure Network Watcher - IP flow verify checks if a packet is allowed or denied to or from a virtual machine. The information consists of direction, protocol, local IP, remote IP, local port, and remote port. If the packet is denied by a security group, the name of the rule that denied the packet is returned. While any source or destination IP can be chosen, IP flow verify helps administrators quickly diagnose connectivity issues from or to the internet and from or to the on-premises environment.