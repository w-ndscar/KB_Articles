### Objective:
***
To install ClamAV antivirus and update the virus database. Then setup On-Access scanning to monitor a folder/mount and move an infected file to quarantine if detected.
***

### Install ClamAV CLI and GUI
***
Update the system by running, 
```bash
sudo apt update && sudo apt upgrade
```

Then install ClamAV using the command,
```bash
sudo apt install clamav clamav-daemon clamtk
```


### ClamAV configuration
***

The config file **clamd.conf** will be in `/etc/clamav`

Open the config file and add the following lines in there:

```conf
TemporaryDirectory /var/temp
OnAccessPrevention yes
OnAccessMountPath /path/to/directory
OnAccessExtraScanning yes
OnAccessMaxFileSize 5000M
```

Then restart the clam daemon by running the command:
```bash
sudo systemctl restart clamd-daemon
```



***