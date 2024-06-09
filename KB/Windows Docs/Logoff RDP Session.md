
### Enter into Powershell Session
***
To enter into a powershell session on another PC, use the following command:
```powershell
Enter-PSSession computer_name -Credential $cred
```
where **computer_name** is the name of the computer/server that you want to connect to and the **-Credential** flag will show a popup to enter the credential that you want to use to connect to that specific computer/server and will be stored on **$cred** variable.
***

### Logoff the Remote Session of a user
***
To logoff a remote user, session ID is needed.
To get the session ID, use the following command:
```powershell
$sessionId = ((quser /server:server_name | Where-Object { $_ -match "user_name" }) -split ' +')[2]
```
where **server_name** is the name of the computer/server and the user_name is the username of the logged in remote user. The session ID will be stored in the $sessionID variable.

Now use the following command to logoff an RDP User Session:
```powershell
Invoke-RDUserLogoff -HostServer server_name -UnifiedSessionId $sessionId
```
where **server_name** is the name of the computer/server.
***

### Restart the RDP Service
***
To restart the RDP Service, enter the following command:
```powershell
Restart-Service -Name TermService -Force
```
The **-Force** flag is required to stop and start the RDP Service
***

