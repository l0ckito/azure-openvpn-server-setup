OpenVPN Server Setup on Azure (Technical)
VM Details

OS: Ubuntu 24.04

Public IP: 134.33.66.43

VM Size: Standard D2s v3

Location: West US 3

Resource Group: openvpn-server_group

1. Update System Packages
sudo apt update && sudo apt -y upgrade

2. Install OpenVPN and Easy-RSA
sudo apt install -y openvpn easy-rsa

3. Create Easy-RSA Directory
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki

4. Build Certificate Authority (CA)
./easyrsa build-ca


Enter a strong CA passphrase and set Common Name (e.g., openvpn-ca).

5. Generate Server Certificate & Key
./easyrsa gen-req server nopass
./easyrsa sign-req server server

6. Generate Diffie-Hellman Parameters
./easyrsa gen-dh

7. Generate TLS Authentication Key
openvpn --genkey secret ta.key

8. Copy Server Files to OpenVPN Directory
sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem ta.key /etc/openvpn/

9. Create OpenVPN Server Configuration
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

10. Start and Enable OpenVPN
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
sudo systemctl status openvpn@server

11. Enable IP Forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo nano /etc/sysctl.conf
# ensure line: net.ipv4.ip_forward=1
sudo sysctl -p

12. Configure UFW Firewall
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status

13. Generate Client Certificate (Client: azure1)
cd ~/openvpn-ca
./easyrsa gen-req client1 nopass
# Common Name: azure1
./easyrsa sign-req client client1

14. Collect Client Keys
mkdir -p ~/client-configs/keys
cp pki/ca.crt pki/issued/client1.crt pki/private/client1.key ta.key ~/client-configs/keys/

15. Create Base Client Config
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

16. Package Client Config into .ovpn
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

ls -l ~/client-configs/azure1.ovpn


The azure1.ovpn file is now ready to import on the client device.
