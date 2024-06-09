### Objective: Extract wim image from a Windows ISO image (Retail and Server)

***

### Extracting the wim file from the ISO

#### Windows Server

* For **Windows Server** the boot.wim and install.wim files can easily be copied
* Mount the ISO and head to the **sources folder**
* Under the **sources** folder, **boot.wim** and **install.wim** can be found

#### Windows 10 & 11

* **Windows 10** and **Windows 11** contains multiple versions (Home, Pro, Pro N and so on) as a package together in an ESD format
* Use the following command to list the different version along with the index number

	`dism /Get-WimInfo /wimfile:filename.esd` 

* Then use the command to get the Windows 10 Pro image.

	`dism /export-image /SourceImageFile:filename.esd /SourceIndex:6 /DestinationImageFile:destinationfolder/filename.wim /Compress:max /CheckIntegrity` 
	
	Note: `/SourceIndex:6` The index number for Windows 10 Pro was 6 which can be obtained by running the previous `/Get-WimInfo` command
***