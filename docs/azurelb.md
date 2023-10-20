---
comments: true
---

# Deploying Website with Azure Load Balancer

## Step 1: Create a Resource Group

1. Sign in to the Azure Portal [https://portal.azure.com](https://portal.azure.com).
2. Click on the "Create a resource" button (green plus sign) in the upper-left corner.
3. Search for "Resource group" and select it.
4. Click the "Create" button.
5. Fill in the details for the resource group:
   - **Subscription:** Choose your Azure subscription.
   - **Resource group:** Enter a unique name for your resource group.
   - **Region:** Select a region for your resource group.
6. Click the "Review + create" button and then "Create" to create the resource group.

???+ info "Resource Group"
    ![Screenshot 2023-10-18 173058.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173058.png)

## Step 2: Create a Network Security Group (NSG) with Inbound Rules

1. In the Azure Portal, search for "Network security group" and select it.
2. Click the "Create" button.
3. Fill in the details for the NSG:
   - **Name:** Enter a unique name for your NSG.
   - **Resource group:** Choose the resource group created in Step 1.
   - **Region:** Select the same region as your resource group.
4. Click the "Review + create" button and then "Create" to create the NSG.

5. After creating the NSG, select it and navigate to the "Inbound security rules" section.

6. Add the following inbound rules:

   - **Rule 1: SSH (Port 22 Inbound)**
     - **Name:** SSH
     - **Priority:** Choose a priority value (e.g., 100)
     - **Source:** Any
     - **Service:** SSH
     - **Action:** Allow

   - **Rule 2: HTTP (Port 80 Inbound)**
     - **Name:** HTTP
     - **Priority:** Choose a priority value (e.g., 200)
     - **Source:** Any
     - **Service:** HTTP
     - **Action:** Allow

   - **Rule 3: ICMP Deny (Block All ICMP Inbound)**
     - **Name:** ICMP Deny
     - **Priority:** Set the priority to 101
     - **Source:** Any
     - **Service:** ICMP
     - **Action:** Deny

7. Save the changes to the NSG.

???+ info "Network Security Group"
    === "NSG"
        ![Screenshot 2023-10-18 173148.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173148.png)
    === "SSH"
        ![Screenshot 2023-10-18 173333.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173333.png)
    === "HTTP"
        ![Screenshot 2023-10-18 173410.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173410.png)
    === "ICMP"
        ![Screenshot 2023-10-20 075809.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-20%20075809.png)

## Step 3: Create a Virtual Network (VNet)

1. In the Azure Portal, search for "Virtual network" and select it.
2. Click the "Create" button.
3. Fill in the details for the VNet:
   - **Name:** Enter a unique name for your VNet.
   - **Resource group:** Choose the resource group created in Step 1.
   - **Region:** Select the same region as your resource group.
4. Configure the address space and subnets for your VNet.
5. Click the "Review + create" button and then "Create" to create the VNet.

???+ info "Virutal Network"
    === "VNet"
        ![Screenshot 2023-10-18 173539.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173539.png)
    === "SubnetA"
        ![Screenshot 2023-10-18 173739.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173739.png)
    === "SubnetB"
        ![Screenshot 2023-10-18 173917.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20173917.png)
    === "IP Addresses"
        ![Screenshot 2023-10-18 174150.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20174150.png)

## Step 4: Create a Load Balancer

1. In the Azure Portal, search for "Load balancer" and select it.
2. Click the "Create" button.
3. Fill in the details for the load balancer:
   - **Name:** Enter a unique name for your load balancer.
   - **Resource group:** Choose the resource group created in Step 1.
   - **Region:** Select the same region as your resource group.
   - Choose the "Internet-facing" or "Internal" load balancer, depending on your requirements.
4. Configure the front-end IP configuration, back-end pools, and health probes as needed.
5. Click the "Review + create" button and then "Create" to create the load balancer.

???+ info "Load Balancer"
    === "Basics"
        ![Screenshot 2023-10-18 174945.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20174945.png)
    === "Frontend IP configuration"
        ![Screenshot 2023-10-18 175111.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175111.png)
    === "Public IP Address (frontend IP configuration)"
        ![Screenshot 2023-10-18 175057.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175057.png)
    === "Backend Pool"
        ![Screenshot 2023-10-18 175306.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175306.png)
    === "Load Balancing Rule"
        ![Screenshot 2023-10-18 175425.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175425.png)
    === "Inbound NAT Rule"
        ![Screenshot 2023-10-18 175619.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175619.png)

## Step 5: Create a Virtual Machine Scale Set (VMSS)

1. In the Azure Portal, search for "Virtual machine scale set" and select it.
2. Click the "Create" button.
3. Fill in the details for the VMSS:
   - **Basics:**
     - **Subscription:** Choose your Azure subscription.
     - **Resource group:** Choose the resource group created in Step 1.
     - **Region:** Select the same region as your resource group.
     - **Name:** Enter a unique name for your VMSS.
   - **Image:**
     - Choose a base image for your virtual machines.
   - **Disks**
      - change the OS disk type to Standard SSD
   - **Networking:**
     - **Virtual network:** Select the VNet created in Step 3.
     - **Subnet:** Choose a subnet within the VNet.
     - **Public IP address:** Depending on your configuration, choose to have a public IP or not.
     - **Load balancer:** Select the load balancer created in Step 4.
   - **Scaling:**
     - Configure the scaling options based on your requirements.
   - **Advanced:**
     - select Enable User Data and input the following command.

```bash
#!/bin/bash

# Update system and install Apache2 and jq
apt-get update -y
apt-get install -y apache2 jq

# Ensure Apache2 is running and enabled on boot
systemctl start apache2
systemctl enable apache2

# Fetch Azure VM metadata
METADATA=$(curl -H Metadata:true -s "http://169.254.169.254/metadata/instance?api-version=2021-01-01")

# Log metadata for debugging purposes
echo "$METADATA" > /tmp/metadata.json

# Extract data from the fetched metadata
local_ipv4=$(echo "$METADATA" | jq -r '.network.interface[0].ipv4.ipAddress[0].privateIpAddress')
az=$(echo "$METADATA" | jq -r '.compute.location')
vm_id=$(echo "$METADATA" | jq -r '.compute.vmId')

# Generate an HTML file with the extracted data
cat <<EOF > /var/www/html/index.html
<!doctype html>
<html lang="en" class="h-100">
<head>
<title>Details for Azure VM</title>
</head>
<body>
<div>
<h1>Azure Instance Details</h1>
<h1>Samurai Katana</h1>

<p><b>Instance Name:</b> $(hostname -f)</p>
<p><b>Instance Private IP Address:</b> ${local_ipv4}</p>
<p><b>Availability Zone:</b> ${az}</p>
<p><b>Virtual Machine ID:</b> ${vm_id}</p>
</div>
</body>
</html>
EOF

# Remove the temporary file
rm /tmp/metadata.json
```

???+ info "VMSS"
    === "Basics pt. 1"
        ![Screenshot 2023-10-18 175823.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175823.png)
    === "Basics pt. 2"
        ![Screenshot 2023-10-18 175837.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175837.png)
    === "Disks"
        ![Screenshot 2023-10-18 175057.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175848.png)
    === "Network Interface"
        ![Screenshot 2023-10-18 175948.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20175948.png)
    === "Netoworking"
        ![Screenshot 2023-10-18 180016.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20180016.png)
    === "Scaling"
        ![Screenshot 2023-10-18 180030.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20180030.png)
    === "Health"
        ![Screenshot 2023-10-18 180039.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20180039.png)
    === "User Data (advanced)"
        ![Screenshot 2023-10-18 180129.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/Screenshot%202023-10-18%20180129.png)

1. Click the "Review + create" button and then "Create" to create the VMSS.
    - copy the public IP address of the VMSS and add http:// to the beginning and paste it into a new tab. your new website should be live.

![azlb.mp4](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/azurelbsc/azlb.gif)

## Congratulations! 🎉

You've successfully completed the lab!

![Congratulations Image](https://www.icegif.com/wp-content/uploads/2023/05/icegif-1098.gif)
