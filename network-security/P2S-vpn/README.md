# VPN
- The aim of using a VPN is to allow secure communication between devices and internal applications running in an Azure VNet over the internet — without using public IPs of VNet resources.
Traffic flowing over the internet is exposed to all kinds of threats. A VPN creates a secure tunnel through the internet.

The VPN tunnel can use different protocols such as OpenVPN, Secure Socket Tunneling, and IKEv2, etc.

Client computers need to authenticate to the VPN to access resources.
This authentication can be done using a certificate, Entra ID, etc.

## There are two types of VPN connections:

Point-to-Site (P2S) — VPN connection is established between an individual device (or devices) and a network.
Example: When a remote employee logs in to their organization’s network from home.

Site-to-Site (S2S) — VPN connection between two networks. This is an always-on connection.
All devices on one network can automatically access the other network without requiring client software on individual devices.
Example: Connection between the networks of an on-premises data center and the cloud or a hybrid cloud setup.

Point-to-Site VPN: Implementation Overview
Deploy a Windows web server with Internet Information Services (IIS).
Test the connection with this server over the internet first, then remove its public IP.
The goal is to connect to this server over the internet using a VPN.

Deploy a VPN Gateway.
The VPN Gateway and its resources are deployed in a dedicated subnet called Gateway Subnet.
The VPN Gateway will have a public IP.

Cost consideration: VPN Gateways are billed per hour and can be expensive.

Authenticate clients.
For lab purposes, we’ll use a self-signed certificate generated using a PowerShell script.

Download and install the VPN client from the VPN Gateway on the client machine to establish the VPN connection.
