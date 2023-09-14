
#### **Objective**: Create a custom Windows 10 Pro image and deploy it using Windows Deployment Services server via PXE

#### Prerequisites:  

- A Windows image file (wim format) is needed. Please refer to this Document on how to [Extract wim from an ISO image](obsidian://open?vault=KB&file=WDS%20Setup)
## Create a config/catalog XML file

- To create a config (XML) file, we need **Windows System Image Manager**. **Windows System Image Manager** is a part of [Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install).  Make sure to choose and install the right version of ADK.
- The config file can be created without the Windows OS image. However, the things that we can customize will be limited. It is recommended to use a Windows OS image (wim file format) while creating the config.
- Open **Windows System Image Manager**. Click **File -> New Answer File**.
- Browse for the Windows OS image (wim format), and click **Ok**.
- Then it'll prompt to create a catalog for that image. Click on **Yes**.
- Now there are a wide varieties of customizations that we can do here. Please refer to these links for the documentations. [Windows System Image Manager Technical Reference](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/wsim/windows-system-image-manager-technical-reference) and [Answer files](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs?view=windows-11)
- For an instance, we do the following customizations: AcceptEula, Skipping EULA and account setup on OOBE (Out-of-the-Box-Experience) and enable Built-in Administrator and set a password.
- At the left bottom side, you'll see your Windows image name and under that two directories: Components and Packages.
- Expand **Components**. Scroll down and find amd64_Microsoft-Windows-Setup-versionnumber and expand it. Then Right click on **UserData** and select **Add Setting to Pass1 windowsPE**.
- Scroll and find amd64_Microsoft-Windows-Shell-Setup-versionnumber and expand it. Right click on **OOBE** and select **Add Setting to Pass7 windowsPE**. Then Right click on **UserAccounts** and select **Add Setting to Pass7 windowsPE**.
- Then at the middle Answer File section, expand **1 windowsPE** and expand amd64_Microsoft-Windows-Setup-versionnumber and select UserData. Then on the right side under UserData Properties sections, set the value of **AcceptEula** to **true**.
- Expand **7 oobeSystem** and expand amd64_Microsoft-Windows-Shell-Setup-versionnumber and select **OOBE** and on OOBE Properties set the following values to **true**: HideEULAPage, HideLocalAccountScreen, HideOEMRegistrationScreen, HideOnlineAccountScreen, HideWirelessSetupinOOBE, SkipMachineOOBE, SkipUserOOBE.
- The answer file is now ready. Click **File -> Save Answer File As** and rename it **unattend.xml**.

### Creating a custom image

- Just do a normal install of Windows OS in a bare metal or a virtual machine.
- Install the necessary and perform the necessary tweaks.
- Set the adjust the boot priority and set **Network Boot** at the top (This step is **very important** for **Sysprep**). Make sure DHCP is available.
- Note that if you boot into the Windows instead of Network Boot after initiating the **Sysprep**, the procedure will fail and you have to wipe and reinstall the Windows OS from the beginning.
- Now copy the answer file (unattend.xml) to C:/Windows/System32/Sysprep folder in the newly install Windows. The xml config file has to be named **unattend.xml** for Sysprep to automatically recognize it.
- Launch **Sysprep.exe** and and select **Enter system Out-of-Bos-Experience (OOBE)** and check the **Generalize** box and choose **Shutdown**.
- Turn the PC back on and boot into the Network (PXE). After WDS has loaded, on the setup page, press **Shift + F10** to open the Administrator command prompt. Mount the REMINST folder using the following command:

	 `net use z: \\IPADDRESSOFWDS\REMINST /user:Username Password`

- Then switch to the mount 'z' drive by typing `z:` and press Enter.
- Capture the installed image using the following command:

	 `dism /Capture-Image /ImageFile:"filename.wim" /CaptureDir:C:\ /Name:"Windows 10 Custom"` 
	 
	Note: `/CaptureDir:` varies. Please make sure the drive letter where the OS resides
- The custom wim image is now ready for the installation.