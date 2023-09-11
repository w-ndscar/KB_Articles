***

## 192.168.56.150

* Admin PC
* Contains software share
* Remoting into PCs and servers
* Not in domain


## WSUS and WDS Setup

### 192.168.56.26 - WSUS and WDS Server

### WSUS

* Manually synchronized
* Uses Group policies to configure updates to domain PCs
* Custom groups are specified in the group policy for managing different networks and PCs

### WDS

* **Windows System Image Manager** is used to create a config file (XML) where the post-deployment properties are specified such as **Skip EULA page, Setup Admin password** and so on
* Custom Windows 10 Pro image is created using **DISM** tool after installing the necessary software and removing bloat and telemetry
* Then the image is fetched via PXE (Network boot) and installed on the client machine