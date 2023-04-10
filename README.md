# AWS to Azure site-to-site VPN solution

---

## I. Architecture:
![VPN Solution Flowchart.png](resources%2FVPN%20Solution%20Flowchart.png)

---

## **II. Components:**

### **A. Azure**
1. **Resource Group**
   * **Name**: azure-rg
2. **Virtual Network**
   * **Basics**:
     * **Resource group**: azure-rg
     * **Virtual network name**: azure-vn
     * **Region**: East US
   * **Networking**:
     * **IP address space**: 172.10.0.0/16
     * Subnet:
       * Name: subnet-01
       * Address prefix: 172.10.1.0/24
3. **VPN network gateway**
   * **Basics**:
     * **Name**: azure-vpn-ngw
     * **SKU** (Stock keeping unit): VpnGw1
     * **Generation**: Generation1
     * **Region**: East US
     * **Virtual network**: azure-vn
     * **Public ip address name**: pubip-vpn-azure-aws
     * **Enable active-active mode**: Disabled
     * **Configure BGP** - Disabled
     * **Public IP Address** - 20.124.200.55 (Should be available upon creation of VPN network gateway)
4. **Local Network gateway**
   * **Basics**:
     * **Name**: azure-aws-lng
     * **Endpoint**: IP address
     * **IP address**: 35.166.40.185 (IPsec tunnel #1 Outside IP Address, Virtual Private Gateway on the aws site-to-site vpn configuration)
     * **Address Space**: 10.10.0.0/16
     * **Region**: East US
5. **Virtual network gateway connections**
   * **Basics**:
     * **Name**: azure-aws-connection-1
     * **Connection type**: Site-to-site(IPsec)
     * **Virtual network gateway**: azure-vpn-ngw
     * **Local network gateway**: azure-aws-lng
     * **Shared Key (PSK)**: (IPsec tunnel #1, Pre-Shared Key)
     * **Use Azure Private IP Address**: No
     * **Enable BGP**: No
     * **IKE Protocol**: IKEv2

     **_**Create connection, at first connection would show as "Unknown", this should change after 5-10mins upon creation_**
#### **High Availability**:

6. **Local Network gateway #2**
   * **Basics**:
     * **Name**: azure-aws-lng
     * **Endpoint**: IP address
     * **IP address**: 35.166.40.185 (IPsec tunnel #2 Outside IP Address, Virtual Private Gateway)
     * **Address Space**: 10.10.0.0/16
     * **Region**: East US
7. **Virtual network gateway connection #2**
   * **Basics**:
     * **Name**: azure-aws-connection-2
     * **Connection type**: Site-to-site(IPsec)
     * **Virtual network gateway**: azure-vpn-ngw
     * **Local network gateway**: azure-aws-lng
     * **Shared Key (PSK)**: (IPsec tunnel #2, Pre-Shared Key)
     * **Use Azure Private IP Address**: No
     * **Enable BGP**: No
     * **IKE Protocol**: IKEv2

### **B. AWS**
1. **Virtual private network (VPC)**
   * **Name**: aws-vpc-01
   * **Region**: us-west-2 (Oregon) 
   * **IPv4 CIDR Block**: 10.10.0.0/16
   * **IPv6 CIDR Block**: No
2. **Subnet**
   * **Name**: subnet-01
   * **Vpc**: aws-vpc-01
   * **Az**: us-west-2a
   * **IPv4 CIDR**: 10.10.1.0/24
3. **Customer gateway**
   * **Name**: azure-cg
   * **Routing**: static
   * **IP Address**: 20.124.200.55 (Azure VPN Network gateway public IP address)
4. **Virtual Private Gateway**
   * **Name**: aws-vpg
   * **Autonomous System Number (ASN)**: Amazon default ASN
   * **VPC**: aws-vpc-01 (Attach to VPC)
5. **Site-to-site VPN connections**
   * **Name**: aws-azure-vpn-connection
   * **Target gateway type**: Virtual Private Gateway
   * **Virtual Private Gateway**: aws-vpg
   * **Customer Gateway**: Existing
   * **Customer gateway ID**: azure-cg
   * **Routing Options**: Static
   * **Static IP Prefixes**: 172.10.1.0/24 (Azure Subnet CIDR)
   * **Tunnel inside ip Version**: IPv4
6. **Route table**
   * **Name**: aws-rtb
   * **Subnet Association**: subnet-01
   * **Routes**
     * **AWS VPG to Azure Subnet**:
       * **Destination**: 172.10.1.0/24 (Azure Subnet)
       * **Target**: aws-vpg (AWS Virtual private gateway)
     * **AWS Internet gateway to internet**:
       * **Destination**: 0.0.0.0/0
       * **Target**: aws-igw
7. **Internet gateway**
   * **Name**: aws-igw
   * **Attached VPC**: vpc-01

## **II. Cost estimates:**
* [AWS site-to-site vpn pricing](https://calculator.aws/#/estimate?id=6bdca12f0f25d4c4f9fe1a03c9d1f2a39d1430b6)
* [Azure site-to-site vpn pricing](https://azure.com/e/0adbc445cba1454cb6db7b0ec7c98b19)