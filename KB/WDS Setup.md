
### WDS

* WDS Role can be added in the server
* Then a folder is specified where the boot, image and other files will go in
* Extract the wim file from the ISO
* For Windows server the boot.wim and install.wim files can easily be copied
* For Windows 10, it contains multiple versions (Home, Pro, Pro N and so on) as a package together in an ESD format
* Use the following command to `dism /Get-WimInfo /wimfile:filename.esd` to list the different version along with the index number
* Then use the command `dism /export-image /SourceImageFile:filename.esd /SourceIndex:6 /DestinationImageFile:destinationfolder/filename.wim /Compress:max /CheckIntegrity` to get the Windows 10 Pro image. Note: `/SourceIndex:6` Specify the correct index number for the specific version
* Once the wim image is obtained, place the image in the WDS Location (Default Location: `C:\Remoteinstall`) 
* Open WDS. Right click on the Boot Images, click Add Install Image and browse for the boot.wim. Give it a name and click Finish
* Then Right click on the Install Images, click Add Install Image. Select an Existing Group or Create an image group and give it a name. Click Next and browser for the install.wim.
* On the next page, uncheck the Use the default name and description and customize the names and make them a bit shorter to fit in the GUI


## Installing the custom image

* On the client machine side, select Network boot on the first priority and boot into it.
* Then click Next on the setup screen and it'll prompt for Username/Password | Note: Username: `FQDN\Username`
* Then select the OS and proceed with the installation