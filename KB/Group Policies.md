****
##### Table of Contents:
- [[#RDP Idle Timeout Logoff|RDP Idle Timeout Logoff]]
- [[#Hide Shutdown and Sleep|Hide Shutdown and Sleep]]
- [[#Login Banner|Login Banner]]
- [[#Enable WinRM|Enable WinRM]]
- [[#RPC WMI WinRM Firewall|RPC WMI WinRM Firewall]]
- [[#Disable Admin Share|Disable Admin Share]]
- [[#Enable Admin Share|Enable Admin Share]]
- [[#High Performance Power Plan|High Performance Power Plan]]
- [[#Set Time Zone|Set Time Zone]]

***

## RDP Idle Timeout Logoff
##### **OU: Domain Computers**

`Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Session Host -> Session Time Limits` 

-  Set time limit for active but idle Remote Desktop Services session -> Enabled: 6 hours.
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
- Always close all applications and log off your office remote computer at the end of every day to ensure the security of your data.
- Don’t forget to restart your Office remote computer every Friday to keep it running smoothly. If your system prompts for a restart due to recent Windows updates, please do so to keep your computer up-to-date and secure
- Save your working files frequently, at least once every 30 minutes, to prevent losing any unsaved work
Thank you for helping to keep our systems secure and running smoothly.
****


## Enable WinRM
##### **OU: Domain Computers, Backup PC**  

- `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> System Services`
- ***Windows Remote Service (WS-Management)*** -> Check “Define this policy setting” and select Automatic
- `Computer Policies -> Preferences -> Control Panel Settings -> Services.` Right click and Select New -> Service
- Service Name: ***WinRM***. Recovery Tab -> First, Second and Subsequent failures -> Select “Restart the service”
- Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Remote Management (WinRM) -> WinRM Service
- Enable ***Allow remote server management through WinRM***
- In IPv4/IPv6 boxes, specify IP addresses or enter * for all IP addresses
- Create Windows Defender Firewall rules allowing WinRM connections on the default ports TCP/5985 and TCP/5986. Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Firewall with Advanced Security -> Windows Firewall with Advanced Security -> Inbound Rules
- Right Click and select New Rule. Select Predefined -> ***Windows Remote Management***
- Finally, Computer Configuration -> Policies -> Windows Components -> Windows Remote Shell -> Enable ***Allow Remote Shell Access***  


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


## Set Time Zone
##### **OU: Domain Computers, Backup PC**  

`Computer Configuration -> Preferences -> Windows Settings -> Registry`

- New -> Registry Wizard
- Select **Local**. Navigate to `HKLM\System\CurrentControlSet\Control\TimeZoneInformation`
- Select all the registry item of **TimeZoneInformation** key.
- Select **TimeZoneKeyName**. Value type: **REG_SZ**. Value data: **India Standard Time**  

***