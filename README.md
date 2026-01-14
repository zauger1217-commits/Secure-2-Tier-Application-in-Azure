# 🔐 Secure 2-Tier Application in Azure

This lab walks through building a **secure 2-tier architecture** in Microsoft Azure using:
- A **public web server VM**
- A **private database server VM**

The goal is to validate **network connectivity, subnet isolation, and basic security controls**.

---

## 🗺️ Architecture Overview

![Azure 2-Tier Architecture](images/architecture-overview.png)

*High-level view of the virtual network, subnets, and virtual machines.*

- **Resource Group:** rg-lab02
- **Virtual Network:** vnet-lab02 (10.0.0.0/16)
- **Subnets:**
  - snet-web (10.0.1.0/24)
  - snet-db (10.0.2.0/24)
- **VMs:**
  - vm-web-01 (Public Web Server)
  - vm-db-01 (Private Database Server)

---

## 📋 Prerequisites

- Active Azure subscription
- Pay-as-you-go billing enabled
- SSH key pair (generated during VM creation)

---

## 🧱 Step 1: Create a Resource Group

![Create Resource Group](images/create-resource-group.png)

*Azure Portal view showing creation of a new resource group.*

1. Go to **Resource Groups**
2. Select **Create**
3. Name: `rg-lab02`
4. Select a region
5. Click **Create**

---

## 🌐 Step 2: Create a Virtual Network

![Virtual Network Setup](images/virtual-network.png)

*Virtual network with web and database subnets configured.*

### Basics Tab
- Resource Group: `rg-lab02`
- Virtual Network Name: `vnet-lab02`

### Security Tab
- Skip

### IP Addresses
- Address space: `10.0.0.0/16`
- Create two subnets:
  - **snet-web**: `10.0.1.0/24`
  - **snet-db**: `10.0.2.0/24`
- Leave all other settings as default

### Tags
- Skip

### Review + Create
- Click **Create**

---

## 🖥️ Step 3: Deploy Virtual Machines

### 🌍 VM1: Public Web Server (vm-web-01)

![Web Server VM](images/web-vm.png)

*Public-facing Ubuntu VM hosting the web tier.*

#### Basics Tab
- Resource Group: `rg-lab02`
- VM Name: `vm-web-01`
- Region: East US (or preferred region)
- Image: **Ubuntu Server 24.04 LTS (x64 Gen2)**
- Size: **Standard_D4s_v3** (4 vCPU, 16 GiB RAM)
- Username: `azureuser`
- Authentication: SSH public key
- Key pair name: `key-lab02`
- Inbound ports allowed: **22 (SSH), 80 (HTTP)**

#### Disks Tab
- Skip

#### Networking Tab
- Ensure Virtual Network, Subnet, and Public IP are auto-filled

#### Management Tab
- Enable **Auto-shutdown**

#### Monitoring / Advanced / Tags
- Skip

#### Review + Create
- Click **Create**
- Save the private SSH key locally

---

### 🗄️ VM2: Database Server (vm-db-01)

![Database Server VM](images/db-vm.png)

*Private VM with no public IP, isolated in the database subnet.*

#### Basics Tab
- Resource Group: `rg-lab02`
- VM Name: `vm-db-01`
- Use existing SSH key stored in Azure
- Key name: `key-lab02`
- Inbound ports allowed: **22 only**

#### Disks Tab
- Skip

#### Networking Tab
- Leave default
- **No Public IP** (important)

#### Monitoring Tab
- Disable **Boot diagnostics**

#### Tags
- Skip

#### Review + Create
- Click **Create**

---

## 🔑 Step 4: Connect to the Web Server

![SSH Connection](images/ssh-connect.png)

*Using SSH to connect securely to the web server.*

### Fix SSH Key Permissions

```bash
chmod 600 <private-key-file-path>
```

### Connect via SSH

```bash
ssh -i <private-key-file-path> azureuser@<vm-web-01-public-ip>
```

You can connect using:
- Azure Portal → **Connect**
- Local terminal or PowerShell

---

## 🔄 Step 5: Connect from Web Server to Database Server

1. Retrieve the **private IP address** of `vm-db-01` from Azure Portal
2. From `vm-web-01`, test connectivity:

```bash
ping <db-private-ip>
```

### Configure Network Security Group (NSG)

![NSG Rule](images/nsg-rule.png)

*Inbound rule allowing traffic from the web subnet to the database subnet.*

1. Open the NSG attached to **vm-db-01**
2. Add a new **Inbound Security Rule**:

| Setting | Value |
|------|------|
| Source | IP Addresses |
| Source IP | Web subnet range (10.0.1.0/24) |
| Service | Custom |
| Destination Port | * |
| Protocol | Any |
| Priority | 100 |
| Name | Allow-Web-Subnet |

This allows traffic **only from the web subnet** to the database server.

---

## 🎯 Lab Purpose

This lab is designed to:
- Validate Azure virtual networking
- Demonstrate subnet isolation
- Practice secure VM deployment
- Test controlled east–west traffic

Optional extensions:
- Install a web application on `vm-web-01`
- Install Microsoft SQL Server or another DB on `vm-db-01`
- Expand into a production-style environment

---

## 🧹 Cleanup (Optional)

To avoid charges, delete the resource group:

```text
rg-lab02
```

This removes all associated resources.

---

✅ Lab Complete
