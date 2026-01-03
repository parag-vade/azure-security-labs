# Azure Virtual WAN

Azure Virtual WAN is primarily used to solve the "spaghetti network" problem in large global enterprises. It replaces complex, unmanageable meshes of individual VNet peerings and VPN gateways with a single, unified "hub-and-spoke" architecture that supports **transitive routing** [web:8][web:12].

- **The Problem:** Network setups in large organizations get complicated. You might have mesh networks with many **VNet peerings** (connecting Azure networks), alongside P2S and S2S VPNs (connecting users and branches), and ExpressRoute connections. Managing security and routing across all these links is difficult [web:12].

- **The Solution:** Azure Virtual WAN simplifies this by providing a unified "Global Transit Network Architecture." It connects all network entities (VNets, Branches, Remote Users) through a central hub managed by Microsoft [web:5][web:10].
    - **Spokes (Azure VNets):** Your VNets connect to the Virtual Hub using a **'Virtual Network Connection'**. Once connected, the virtual hub can provide transit routing between VNets in **Standard Virtual WAN**. (In Basic tier, transitive routing is not available.)
    - **Branches:** (On-prem networks, datacenters, HQs, Branch offices / other cloud networks) connect to the Virtual Hub using **Site-to-Site (S2S) VPN** (via Virtual Private Gateway) or **ExpressRoute** (via ExpressRoute gateway in the hub) [web:10][web:5].
    - **Users:** Individual user devices (laptops) are connected via **Point-to-Site (P2S) / User VPN**.
        - *"User VPN"* is simply the name Azure Virtual WAN uses for Point-to-Site (P2S) VPN.
        - Users are "endpoints" that need secure access to Spokes (apps) or Branches (internal file servers) [web:68][web:65].

## Virtual WAN SKUs

The SKU is set on the **Virtual WAN** resource (the top-level container), but it dictates the capabilities of the **Virtual Hubs** inside it. When you create the top-level "Virtual WAN" resource, you choose "Basic" or "Standard" [web:8][web:23].

- If your Virtual WAN is **Basic**, *all* hubs inside it will be Basic (**No** transitive routing, limited to S2S VPN only).
- If your Virtual WAN is **Standard**, *all* hubs inside it are Standard (capable of ExpressRoute, P2S, VNet-to-VNet transit, etc.).
- **Note:** You cannot mix and match (e.g., you cannot have a "Basic" hub inside a "Standard" WAN).

## Virtual Hubs - How many are needed?

- **The Golden Rule:** Typically, deploy **one** Virtual Hub per Azure region. This is the most common real-world scenario. A single hub is designed to handle an entire region's traffic. It can scale to **20 Gbps** for S2S VPN throughput and supports up to **500 VNet connections** [web:130][web:128].
- **Global Connectivity:** To connect resources globally—such as VNets in the USA, Europe, and Asia, data centers in Silicon Valley and Mumbai, and users worldwide—you deploy one hub in **each** of these regions.
    - These hubs are **automatically connected** by the Microsoft Global Backbone (Standard SKU).
    - *Example:* A user in India can access files in a USA data center by hopping from the Indian Hub to the USA Hub [web:29][web:136].
- **Multiple Hubs in One Region:** Rarely, you may need multiple hubs in a single region. This is required only if you exceed the limits of a single hub (e.g., needing more than **500 VNet connections** or exceeding the **20 Gbps S2S VPN** / **50 Gbps Hub Router** throughput limits) [web:149][web:128].

## Scale Units (Capacity)

Scale Units define the **size/throughput** of your VPN Gateway or ExpressRoute Gateway.

- In Azure Virtual WAN, you do not select a specific "VM size" (like `Standard_D2s_v3`) for your gateways. Instead, you purchase capacity in blocks called **Scale Units**. This abstraction simplifies capacity planning. You don't need to worry about CPU or memory; you just buy the "speed" you need.
- **For VPN Gateway (P2S or S2S VPN):** 1 Scale Unit = **500 Mbps** capacity.
- **For ExpressRoute Gateway:** 1 Scale Unit = **2 Gbps** capacity.

**Example Calculation (AZ-500 Key Detail):**
If you need **2 Gbps** of throughput:
- For **VPN**, you need **4 Scale Units** (4 x 500 Mbps).
- For **ExpressRoute**, you only need **1 Scale Unit** (1 x 2 Gbps).

- **Billing:** You pay for the **provisioned capacity**, not just the used capacity. If you provision 10 Scale Units (5 Gbps) but only push 100 Mbps of traffic, you are still billed for the full 10 Scale Units per hour [web:30].
- **Redundancy:** Even with **1 Scale Unit**, Microsoft automatically deploys **2 active instances** behind the scenes for redundancy (Active-Active) [web:10].

## Hub Routing Infrastructure Units

This defines the capacity of the **Virtual Hub Router** itself (the component that routes traffic between VNets).

- You increase this if you have thousands of VMs. The default hub supports ~2,000 VMs. If you need to connect more VMs, you buy "Hub Infrastructure Units" to scale the router up to **50 Gbps** [web:10][web:22].

## Workflow

1.  **Create Virtual WAN Resource:** The top-level container (select Basic or Standard SKU here).
2.  **Create Virtual Hub:** The central router/connection point managed by Microsoft.
    - *Option:* Deploy **Azure Firewall** inside to make it a **Secured Virtual Hub**.
3.  **Connect Spokes (VNets):** Create **Virtual Network Connections** to attach workload VNets to the Hub.
4.  **Connect Branches (S2S):** Connect on-prem sites, AWS VPCs, or GCP VPCs via S2S VPN.
5.  **Connect Users:** Configure User VPN (P2S).
6.  **Routing:** The Hub handles routing automatically. In **Standard SKU**, it provides transitive connectivity (e.g., User -> Hub -> VNet) without manual route tables.
