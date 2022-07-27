# Why use VPN

 A Virtual Private Network (VPN) is a significant step in securing communication. It's a requirement in compliance with many standards such as HIPAA, NIST, ... etc.
VPN achieves the encryption of data in-transit or in-motion by creating a tunnel through which, data is transfered back and forth between the two parties of the tunnel.
For an attacker from the outside, the data will look Gubberish, nothing logical. VPN uses mutual authentication which means that each party in the communication is able to authenticate the other party to make sure the other party is who they claim to be. This is implemented by certificate authentication.


# Azure VPN
VPN has different protocols to be built upon. It also can be built on different layers of the OSI model. For example, IPSec works in layer 4 of the OSI model (Transport), whereas L2TP works in layer 2 (Data link) and OpenVPN works in layer 3 (Network). The steps needed to create a VPN in Azure follows:

# VPN Steps:

i. create a virtual gateway 
ii. insert the root certificate
iii. download the client application
iv. install a leaf certificate





