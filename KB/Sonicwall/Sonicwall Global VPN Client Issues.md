### Common issues
***
##### VPN status shows Connecting/Authenticating for a long time:
Restart the PC once and try again. If possible, restart the router/gateway.
If the issue persists, reinstall the Global VPN Client application. Check the box: **Delete the settings** and uncheck the box: **Retain the MAC Address**

***
##### "Phase 1 died" message on Logs
If there is a message "Phase 1 died" on the Logs, go to VPN Properties -> Select Peers tab -> Select the Peer and click Edit.
Under Networking section, select "Forced On" on **NAT Traversal**
![[NAT Traversal.png]]

***
#### Stuck in a Preshared key loop
If the correct preshared key is entered and you receive the error: "The preshared key appears to be incorrect", then go to VPN Properties and check the box: **Restrict the size of the first ISAKMP packet sent**
![[ISAKMP.png]]

***