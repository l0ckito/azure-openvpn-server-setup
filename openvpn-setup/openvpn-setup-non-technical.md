Setting Up OpenVPN on Azure (Non-Technical)

This guide explains how we set up a secure VPN server on an Azure virtual machine. The VPN allows clients to connect securely to the server.

1. Prepare the Server

We began by updating the system and installing the necessary software to run the VPN. This ensures that the server is running the latest versions of the software and security patches. The necessary software includes OpenVPN and Easy-RSA, which are used to create secure connections.

2. Create Security Certificates

We created a "Certificate Authority" (CA), which is like a trusted digital ID card for the server and clients. The CA helps ensure that communication between the server and clients is secure. Using Easy-RSA, we generated a key for the server and a certificate to verify its identity.

3. Configure the Server

We set up the OpenVPN server by creating a configuration file. This file specifies:

Port and Protocol: We decided on using port 1194 and the UDP protocol.

Encryption Certificates: We used the certificates we generated to ensure secure communication between the server and clients.

Network Settings: The VPN network is set up to use the private IP range 10.8.0.0/24, which allows multiple clients to connect securely.

Security Options: Additional security features were set, including encryption methods and access control to ensure safe communication.

4. Start the VPN Service

We started the VPN service, ensuring it runs automatically every time the server restarts. This means the VPN will always be ready to accept client connections without needing to be manually started each time.

5. Enable Internet Access Through VPN

We configured the server to allow traffic from connected VPN clients to reach the internet. This involved enabling IP forwarding on the server, so it can route data between the VPN clients and the internet.

6. Configure Firewall

To allow secure connections, we configured the server's firewall to accept incoming connections on port 1194 (the VPN port) and port 22 for SSH connections. This allows the server to communicate with clients and administrators safely.

7. Create a Client Connection

We created a secure certificate for a client named azure1. This certificate helps the client authenticate securely when connecting to the VPN server. After generating the client certificate, we packaged all the necessary information, including the certificate and keys, into a single file called azure1.ovpn.

This .ovpn file contains all the details needed to connect to the VPN. It can be imported into a VPN application on a computer or phone to securely access the server.

8. Verify and Document

We verified the following to ensure everything was set up correctly:

VPN Service is Running: We checked that the VPN service was active and functioning properly.

Firewall is Configured Correctly: We made sure that the firewall rules were set up to allow the necessary connections.

Client File Exists: We confirmed that the azure1.ovpn client file was created and is ready for use.

Screenshots of each step were saved for future reference and documentation purposes.

Conclusion

The VPN server is now fully set up and ready to use. The client azure1 can securely connect to the VPN using the azure1.ovpn file. All necessary steps have been documented for future reference.
