Setting Up OpenVPN on Azure (Non-Technical)

This guide explains how we set up a secure VPN server on an Azure virtual machine. The VPN allows clients to connect securely to the server.

---

## 1. Prepare the Server
We first updated the system and installed the necessary software to run the VPN.

---

## 2. Create Security Certificates
We created a "Certificate Authority" — a secure digital identity that signs all keys and certificates. This is like creating a trusted ID card system for the server and clients.

---

## 3. Configure the Server
We set up the OpenVPN server with a configuration file, specifying:
- Which port and protocol to use
- Certificates for encryption
- Network settings for VPN clients
- Security options

---

## 4. Start the VPN Service
We started the VPN software and ensured it runs automatically if the server restarts.

---

## 5. Enable Internet Access Through VPN
We allowed the server to forward traffic from VPN clients to the internet.

---

## 6. Configure Firewall
We made sure the server’s firewall allows VPN connections (port 1194) and SSH connections.

---

## 7. Create a Client Connection
We generated a secure certificate for a client named `azure1` and packaged all necessary files into a single `.ovpn` file.  

This `.ovpn` file can be imported into a VPN application on a computer or phone to connect to the VPN.

---

## 8. Verify and Document
We verified:
- VPN service is running
- Firewall is configured correctly
- Client file exists

Screenshots of these steps are saved for reference.

> The VPN server is now ready, and the client `azure1` can securely connect using the `azure1.ovpn` file.
