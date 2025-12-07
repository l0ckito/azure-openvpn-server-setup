# OpenVPN Setup on Azure - Detailed Report

## Overview
This document provides a detailed guide on setting up an OpenVPN server on a Microsoft Azure Virtual Machine (VM), including all steps and commands used during the setup process. The instructions are split into two sections: a technical format for advanced users and a non-technical format for easier understanding.

## Technical Report (Commands and Configuration)

### Step 1: Install Azure CLI and Configure Python Environment

1. **Create a Python Virtual Environment**:

   ```bash
   python3 -m venv azcli-env
Activate Virtual Environment:

bash
Copy code
source azcli-env/bin/activate
Install Azure CLI:

bash
Copy code
pip install azure-cli
Verify Installation:

bash
Copy code
az --version
Step 2: Log into Azure CLI
Login to Azure:

bash
Copy code
az login --use-device-code
Check Account Details:

bash
Copy code
az account show
Step 3: Set up the Resource Group and Network
Create Resource Group:

bash
Copy code
az group create --name openvpn-lab --location eastus
Create Virtual Network and Subnet:

bash
Copy code
az network vnet create --resource-group openvpn-lab --name openvpn-vnet --subnet-name openvpn-subnet --address-prefix 10.0.0.0/16 --subnet-prefix 10.0.0.0/24
Create Public IP Address:

bash
Copy code
az network public-ip create --resource-group openvpn-lab --name openvpn-public-ip --sku Standard --allocation-method Static
Create Network Interface and Attach Public IP:

bash
Copy code
az network nic create --resource-group openvpn-lab --name openvpn-nic --vnet-name openvpn-vnet --subnet openvpn-subnet --public-ip-address openvpn-public-ip
Step 4: Create Virtual Machine (Ubuntu 22.04)
Create VM in East US:

bash
Copy code
az vm create --resource-group openvpn-lab --name openvpn-vm --size Standard_D2s_v3 --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys --public-ip-address openvpn-public-ip --location eastus
Check VM Status:

bash
Copy code
az vm show --resource-group openvpn-lab --name openvpn-vm
Step 5: OpenVPN Setup on the VM
Update and Install OpenVPN & Easy-RSA:

bash
Copy code
sudo apt update && sudo apt upgrade -y
sudo apt install openvpn easy-rsa -y
Set Up Easy-RSA Directory:

bash
Copy code
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
Edit Vars File for CA Configuration:

bash
Copy code
nano vars
Build Certificate Authority:

bash
Copy code
source vars
./clean-all
./build-ca
Generate Server Certificate:

bash
Copy code
./build-key-server server
Non-Technical Report (Simplified Overview)
Step 1: Setting Up the Azure CLI and Python Environment
Install Azure CLI: The Azure CLI was installed within a dedicated environment on the system. The Azure CLI is a tool used to manage Azure services from the command line.

Log into Azure: The login was performed using a secure device code.

Verify Account Details: The account details were confirmed to ensure successful login.

Step 2: Create Network and Virtual Resources
Create a Resource Group: A container was created within Azure to hold all resources related to this project.

Set Up Virtual Network: A virtual network was created to allow different resources within the resource group to communicate securely.

Create a Public IP Address: A public IP address was reserved for the OpenVPN server so that external devices can connect to it.

Create a Network Interface: The public IP was connected to the Virtual Machine to allow internet access.

Step 3: Create the Virtual Machine (VM)
Set Up the Virtual Machine: The VM was created with Ubuntu Linux installed. This will be the server that runs the OpenVPN software.

Check VM Status: The VM's status was verified to ensure everything is functioning properly.

Step 4: Install OpenVPN on the VM
Update the System and Install OpenVPN: The system was updated, and OpenVPN, along with tools needed to create secure certificates (Easy-RSA), was installed.

Create the Certificate Files: OpenVPN requires certificates to secure the connections. These were generated on the VM.

Configure the Server: The server's certificate was generated, which will allow it to securely manage incoming VPN connections.

This guide provides a basic understanding of how to set up a VPN server on Microsoft Azure using OpenVPN.
