Azure OpenVPN Server Setup Report (Technical)
VM Details

OS: Ubuntu 24.04

Public IP: 134.33.66.43

VM Size: Standard D2s v3

Location: West US 3

Resource Group: openvpn-server_group

Step 1: Install Azure CLI and Configure Python Environment

(This step is for setting up the Azure CLI, if needed for automation. Skip this if you're manually configuring the resources in the Azure Portal.)

Create a Python Virtual Environment:

python3 -m venv azcli-env


Activate Virtual Environment:

source azcli-env/bin/activate


Install Azure CLI:

pip install azure-cli


Verify Installation:

az --version

Step 2: Log into Azure CLI

(Only necessary if using Azure CLI to automate the resource creation process.)

Login to Azure:

az login --use-device-code


Check Account Details:

az account show

Step 3: Set up the Resource Group and Network

(These steps create the network and resources on Azure.)

Create Resource Group:

az group create --name openvpn-server_group --location westus3


Create Virtual Network and Subnet:

az network vnet create --resource-group openvpn-server_group --name openvpn-vnet --subnet-name openvpn-subnet --address-prefix 10.0.0.0/16 --subnet-prefix 10.0.0.0/24


Create Public IP Address:

az network public-ip create --resource-group openvpn-server_group --name openvpn-public-ip --sku Standard --allocation-method Static


Create Network Interface and Attach Public IP:

az network nic create --resource-group openvpn-server_group --name openvpn-nic --vnet-name openvpn-vnet --subnet openvpn-subnet --public-ip-address openvpn-public-ip

Step 4: Create Virtual Machine (Ubuntu 24.04)

Create VM in West US 3:

az vm create --resource-group openvpn-server_group --name openvpn-vm --size Standard_D2s_v3 --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys --public-ip-address openvpn-public-ip --location westus3


Check VM Status:

az vm show --resource-group openvpn-server_group --name openvpn-vm

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

Step 6: Configure OpenVPN Server

Generate Diffie-Hellman Parameters:

./build-dh


Generate TLS Authentication Key:

openvpn --genkey secret ta.key


Copy Server Files to OpenVPN Directory:

sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem ta.key /etc/openvpn/


Create OpenVPN Server Configuration:

sudo nano /etc/openvpn/server.conf


Content:

port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0
topology subnet
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
log-append /var/log/openvpn.log
verb 3

Step 7: Start and Enable OpenVPN

Start OpenVPN:

sudo systemctl start openvpn@server


Enable OpenVPN to Start on Boot:

sudo systemctl enable openvpn@server


Check OpenVPN Status:

sudo systemctl status openvpn@server

Step 8: Enable IP Forwarding

Enable IP Forwarding Temporarily:

sudo sysctl -w net.ipv4.ip_forward=1


Make IP Forwarding Permanent:

sudo nano /etc/sysctl.conf


Ensure the line:

net.ipv4.ip_forward=1


Apply Changes:

sudo sysctl -p

Step 9: Configure UFW Firewall

Allow OpenVPN and SSH Connections:

sudo ufw allow 1194/udp
sudo ufw allow OpenSSH


Enable UFW:

sudo ufw enable


Check UFW Status:

sudo ufw status

Step 10: Generate Client Certificate (Client: azure1)

Generate Client Certificate:

cd ~/openvpn-ca
./easyrsa gen-req client1 nopass


Common Name: azure1

Sign the Client Certificate:

./easyrsa sign-req client client1

Step 11: Collect Client Keys

Collect the Client Keys and Certificates:

mkdir -p ~/client-configs/keys
cp pki/ca.crt pki/issued/client1.crt pki/private/client1.key ta.key ~/client-configs/keys/

Step 12: Create Base Client Config

Create Base Client Config File:

nano ~/client-configs/base.conf


Content:

client
dev tun
proto udp
remote 134.33.66.43 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
verb 3

Step 13: Package Client Config into .ovpn

Package Client Configuration into .ovpn:

cd ~/client-configs
cat base.conf \
    <(echo -e '<ca>') \
    keys/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    keys/client1.crt \
    <(echo -e '</cert>\n<key>') \
    keys/client1.key \
    <(echo -e '</key>\n<tls-auth>') \
    keys/ta.key \
    <(echo -e '</tls-auth>') \
    > azure1.ovpn


Verify the .ovpn File:

ls -l ~/client-configs/azure1.ovpn


The azure1.ovpn file is now ready to import on the client device.
