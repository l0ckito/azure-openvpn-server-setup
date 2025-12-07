Technical Report (commands and configuration)
# Azure OpenVPN Server Setup Report (Technical)

## Step 1: Install Azure CLI and Configure Python Environment

1. **Create a Python Virtual Environment**:

   ```bash
   python3 -m venv azcli-env


Activate Virtual Environment:

source azcli-env/bin/activate


Install Azure CLI:

pip install azure-cli


Verify Installation:

az --version

Step 2: Log into Azure CLI

Login to Azure:

az login --use-device-code


Check Account Details:

az account show

Step 3: Set up the Resource Group and Network

Create Resource Group:

az group create --name openvpn-lab --location eastus


Create Virtual Network and Subnet:

az network vnet create --resource-group openvpn-lab --name openvpn-vnet --subnet-name openvpn-subnet --address-prefix 10.0.0.0/16 --subnet-prefix 10.0.0.0/24


Create Public IP Address:

az network public-ip create --resource-group openvpn-lab --name openvpn-public-ip --sku Standard --allocation-method Static


Create Network Interface and Attach Public IP:

az network nic create --resource-group openvpn-lab --name openvpn-nic --vnet-name openvpn-vnet --subnet openvpn-subnet --public-ip-address openvpn-public-ip

Step 4: Create Virtual Machine (Ubuntu 22.04)

Create VM in East US:

az vm create --resource-group openvpn-lab --name openvpn-vm --size Standard_D2s_v3 --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys --public-ip-address openvpn-public-ip --location eastus


Check VM Status:

az vm show --resource-group openvpn-lab --name openvpn-vm

Step 5: OpenVPN Setup on the VM

Update and Install OpenVPN & Easy-RSA:

sudo apt update && sudo apt upgrade -y
sudo apt install openvpn easy-rsa -y


Set Up Easy-RSA Directory:

make-cadir ~/openvpn-ca
cd ~/openvpn-ca


Edit Vars File for CA Configuration:

nano vars


Build Certificate Authority:

source vars
./clean-all
./build-ca


Generate Server Certificate:

./build-key-server server
