***

### Windows Deployment Services
***

WDS Role can be added in the server. Then specify a folder where the boot, image and other files will reside.

Refer to the [Windows Deployment Services](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj648426(v=ws.11)) article from Microsoft for more info.

#### Extracting the wim file from the ISO
***

* For **Windows Server** the boot.wim and install.wim files can easily be copied
* **Windows 10** contains multiple versions (Home, Pro, Pro N and so on) as a package together in an ESD format
* Use the following command to list the different version along with the index number

	`dism /Get-WimInfo /wimfile:filename.esd` 

* Then use the command to get the Windows 10 Pro image (wim).

	`dism /export-image /SourceImageFile:filename.esd /SourceIndex:6 /DestinationImageFile:destinationfolder/filename.wim /Compress:max /CheckIntegrity` 
	
	**Note**: `/SourceIndex:6` Specify the correct index number for the specific version

* Copy the exported wim image to the WDS Location (Default Location: `C:\Remoteinstall`) 

#### Adding the image files in Windows Deployment Services

* Open WDS. Right click on the **Boot Images**, click **Add Install Image** and browse for the *boot.wim*. Give it a name and click Finish
* Then Right click on the **Install Images**, click **Add Install Image**. Select an Existing Group or Create an image group and give it a name. Click Next and browser for the *install.wim*.
* On the next page, **uncheck** the **Use the default name and description** and customize the names and make them a bit shorter to fit in the GUI
* Then click Finish
* It is now safe to delete the wim files that you've copied to the Remoteinstall folder as the image files are added under Boot and Images directories


## Installing the custom image
***

* On the client machine side, select Network boot on the first priority and boot into it.
* Then click Next on the setup screen and it'll prompt for Username/Password | Note: Username: `FQDN\Username`
* Then select the OS and proceed with the installation