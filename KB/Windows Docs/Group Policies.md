***
##### Table of Contents:
- [[#RDP Idle Logout]]
- [[#Hide Shutdown and Sleep]]
- [[#Login Banner]]
- [[#Enable WinRM]]
- [[#RPC WMI WinRM Firewall]]
- [[#Disable Admin Share]]
- [[#Enable Admin Share]]
- [[#High Performance Power Plan]]
- [[#IST Time Zone]]
- [[#WSUS Domain Policy]]
- [[#Block IP Addr Outbound Firewall]]
- [[#Set Local Administrators]]
- [[#Remote Desktop Shadow WP (with User's Permission)]]
- [[#Remote Desktop Shadow WOP (without User's Permission)]]
- [[#RDP Optimization]]

***

## RDP Idle Logout
##### **OU: Domain Computers**

`Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Session Host -> Session Time Limits` 

-  Set time limit for disconnected sessions -> Enabled: 6 hours.
-  End session when time limits are reached -> Enabled


## Hide Shutdown and Sleep
##### **OU: Domain Computers, Backup PC**

`Computer Configuration -> Preferences -> Windows Settings -> Registry`

**Hide Shutdown:**
- New -> Registry Item. Action: Update
- Hive: HKEY_LOCAL_MACHINE
- Key path: `SOFTWARE\Microsoft\PolicyManager\default\Start\HideShutDown` 
- Value Name: value
- Value Type: REG_DWORD
- Value data: 1

**Hide Sleep:**
- New -> Registry Item. Action: Update
- Hive: HKEY_LOCAL_MACHINE
- Key path: `SOFTWARE\Microsoft\PolicyManager\default\Start\HideSleep`
- Value Name: value
- Value Type: REG_DWORD
- Value data: 1  


## Login Banner
##### **OU: Domain Computers**  

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> Security Options`  

- Set “Interactive logon: Message title for users attempting to log on” to “Enabled” and configure the title to “Important Computer Maintenance Reminders”.
- Set “Interactive logon: Message text for users attempting to log on” to “Enabled” and configure the text to:
****
```
- Always close all applications and log off your office remote computer at the end of every day to ensure the security of your data.
- Don’t forget to restart your Office remote computer every Friday to keep it running smoothly. If your system prompts for a restart due to recent Windows updates, please do so to keep your computer up-to-date and secure
- Save your working files frequently, at least once every 30 minutes, to prevent losing any unsaved work
	Thank you for helping to keep our systems secure and running smoothly.
```
****


## Enable WinRM
##### **OU: Domain Computers, Backup PC**  

- `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> System Services`
- ***Windows Remote Management (WS-Management)*** -> Check “Define this policy setting” and select Automatic
- `Computer Configuration -> Preferences -> Control Panel Settings -> Services.` Right click and Select New -> Service
- Service Name: ***WinRM***. Service Action: ***Start Service***. Recovery Tab -> First, Second and Subsequent failures -> Select “Restart the service”
- `Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Remote Management (WinRM) -> WinRM Service`
- Enable ***Allow remote server management through WinRM***
- In IPv4/IPv6 boxes, specify IP addresses or enter * for all IP addresses
- Then, Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Remote Shell -> Enable ***Allow Remote Shell Access***
- Create Windows Defender Firewall rules allowing WinRM connections on the default ports TCP/5985 and TCP/5986. Note: This policy is already added in **RPC WMI WinRM Firewall**. Leaving the steps below for a reference
- Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Firewall with Advanced Security -> Windows Firewall with Advanced Security -> Inbound Rules
- Right Click and select New Rule. Select Predefined -> ***Windows Remote Management***


## RPC WMI WinRM Firewall
##### **OU: Domain Computers, Backup PC**  

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Firewall with Advanced Security -> Windows Firewall with Advanced Security -> Inbound Rules`

- **RPC:** New Rule -> Predefined -> **Remote Scheduled Task Management**
- **WMI:** New Rule -> Predefined -> **Windows Management Instrumentation** 
- **WinRM:** New Rule -> Predefined -> **Windows Remote Management**
- **Admin Share:** New Rule -> Predefined -> **File and Printer Sharing**  



## Disable Admin Share
##### **OU: Domain Computers, Backup PC**  

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Firewall with Advanced Security -> Windows Firewall with Advanced Security -> Inbound Rules`

**Admin Share:** New Rule -> Predefined -> **File and Printer Sharing**
Select All and click **Block the connection**  


## Enable Admin Share
##### **OU: Domain Computers, Backup PC**  

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Firewall with Advanced Security -> Windows Firewall with Advanced Security -> Inbound Rules`

**Admin Share:** New Rule -> Predefined -> **File and Printer Sharing**
Select All and click **Allow the connection**  


## High Performance Power Plan
##### **OU: Domain Computers, Backup PC**

`Computer Configuration -> Policies -> Administrative Templates -> System -> Power Management`

Enable “Select an active power plan” and select **High Performance**  


## IST Time Zone
##### **OU: Domain Computers, Backup PC**  

`Computer Configuration -> Preferences -> Windows Settings -> Registry`

- New -> Registry Wizard
- Select **Local**. Navigate to `HKLM\System\CurrentControlSet\Control\TimeZoneInformation`
- Select all the registry item of **TimeZoneInformation** key.
- Select **TimeZoneKeyName**. Value type: **REG_SZ**. Value data: **India Standard Time**  


## WSUS Domain Policy
##### **OU: Domain Computers, Backup PC** 

`Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Update`

- Set **Configure Automatic Updates** to **Enabled**. Under **Configure automatic updating**, select '3. Auto download and schedule the install'. Configure the schedule options accordingly.
- Set **Specify intranet Microsoft update service location** to **Enabled**. Set the URL/FQDN of the WSUS Server along the with the port number. 8530 for http and 8531 for https.
- Enable Automatic update detection frequency and set it to 1 hour(s).
- Set **Enable client side targeting** to **Enabled** and specify the group name on the WSUS Server.
- Set **Allow Automatic updates immediate installation** to **Enabled**.


## Block IP Addr Outbound Firewall
##### OU: Domain Computers

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Defender Firewall with Advanced Security -> Windows Defender Firewall with Advanced Security -> Outbound Rules`

- Right click and click **New Rule** and give it a name
- Click on **Scope** Tab and under **Remote IP address**, select **These IP addresses** and add the desired IP addresses to the block the RDP access
- Certain ports can also be blocked by selecting the **Protocols and Ports** tab and select the appropriate protocol (say TCP), and select Specific Ports from the dropdown and enter the ports to block
- These are the blacklisted IPs and ports. If this policy is applied to an OU, then the computers in that OU will not be able to remote into or access the specified IPs and ports.  


## RDP_Policy
##### OU: Domain Computers

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> User Rights Assignment`

- Right click on **Allow log on through Remote Desktop Services** and click **Properties**
- Check the box **Define these policy settings** and select **Add User or Group** and select the desired users and groups


## Set Local Administrators

##### OU: Domain Computers

`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Restricted Groups`

- Right Click and Select "Add Group"
- Select Browse and add the Administrators Group and click OK
- Double Click on Administrators and select Add on "Members of the group" section
- Browse and find your security group that you want to grant the Local Admin privilege

## Remote Desktop Shadow WP (with User's Permission) 
***

`Computer Configuration -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Desktop Session Host -> Connections`

- Edit the option "Set rules for remote control of Remote Desktop Services user sessions" and click **Enabled**
- Under **Options**, select **Full Control with user's permission** and click OK

## Remote Desktop Shadow WOP (without User's Permission) 
***

`Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Desktop Session Host -> Connections`

- Edit the option "Set rules for remote control of Remote Desktop Services user sessions" and click **Enabled**
- Under **Options**, select **Full Control without user's permission** and click OK


## Private DNS Policy

`Computer Configuration -> Policies -> Administrative Templates -> Network -> DNS Client`

- Edit **DNS Servers** and click **Enabled**
- Specify at least one DNS server address. Note that the DNS address are space delimited.

## RDP Optimization

`Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Desktop Session Host -> Remote Session Environments`

- Edit "Configure compression for RemoteFX data" and select **Enabled**
- Under RDP compression algorithm, select **Balances memory and network bandwith**
- Edit "Use hardware graphics adapters for all Remote Desktop Services sessions" and select **Enabled**
- Edit "Configure H.264/AVC hardware encoding for Remote Desktop Connections" and select **Enabled**
- Edit "Allow desktop composition for remote desktop sessions" and select **Disabled**
- Edit "Do not allow font smoothing" and select **Disabled**
