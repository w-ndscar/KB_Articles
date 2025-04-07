In this article, we'll look at the setup of 3 types backup solutions. We'll set them up in Ubuntu Server 24.04

Types of Backup:
- Sync
- Deduplicate backup
- Full file and image backup

***

### Sync - Syncthing

- The synchronized backup is achieved using [Syncthing](https://syncthing.net/).
- It offers peer-to-peer sync. Folders can be synced to multiple location at the same time and Syncthing has a nice Web GUI
- The sync duration can be scheduled.
- Syncthing can be downloaded for [here](https://syncthing.net/downloads/)

For Debian/Ubuntu, install using the apt repository
```
sudo apt install syncthing
```

After the installation completes, open the web interface by navigating to the URL http://localhost:8384. By default, the web interface would only open on localhost. If you would like to open the Web GUI from any other machine, go to `~/.local/state/syncthing`, and open **config.xml**. Under `<gui>` section, change the `<address>` from `127.0.0.1:8384` to `0.0.0.0:8384`. 
Make sure to setup proper networking and set a strong password for the Web GUI.
It is also recommended to run syncthing as a regular user. 
```
sudo systemctl start syncthing@username.service
```

Refer to [Syncthing Docs](https://docs.syncthing.net/index.html) for more information.

***

### De-duplicate Backup - Backrest

- [Backrest]() is being using which is a wrapper on [Restic](https://restic.net/) which offers a Web GUI. Restic is a command-line backup utility.  It offers chunk-based incremental backup.
- Offers high speed backup and uses **zstd** compression for best and fast compression
- Download the latest version of Backrest from [here](https://github.com/garethgeorge/backrest/releases)

Install using the Install Script
```
mkdir backrest && tar -xzvf backrest_Linux_x86_64.tar.gz -C backrest
cd backrest && sudo ./install.sh
```

Once the installation is done, Backrest can be accessed from http://localhost:9898 by default. If you would like to access the Web GUI from any other machine, run
```
sudo nano /etc/systemd/system/backrest.service
```
and add the following under `Service`
```
[Service]
Environment="BACKREST_PORT=0.0.0.0:9898"
```

Then restart Backrest service by running,
```
sudo systemctl restart backrest.service
```
***

### Full File and Image Backup - URBackup

- [URBackup](https://www.urbackup.org/) offers both Incremental (De-duplicate) and Full Backup
- It is a Server and Client model where a backup server is configured and the client application is installed on the client which is going to be backed up

##### URBackup Server

To install the URBackup Server on Ubuntu, run the following:
```
sudo add-apt-repository ppa:uroni/urbackup
sudo apt-get update
sudo apt-get install urbackup-server
```

If PPA doesn't get the package, try the following:
```
echo 'deb http://download.opensuse.org/repositories/home:/uroni/xUbuntu_24.04/ /' | sudo tee /etc/apt/sources.list.d/home:uroni.list

curl -fsSL https://download.opensuse.org/repositories/home:uroni/xUbuntu_24.04/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/home_uroni.gpg > /dev/null

sudo apt update

sudo apt install urbackup-server
```

Once the installation is done, the Web GUI can be accessed on http://localhost:55414 by default or by http://server-ip:55414.

##### URBackup Client

To install the URBackup Client on the Ubuntu Desktop, download the [Linux Binary](https://www.urbackup.org/download.html#linux_all_binary) and install it by running the following:
```
TF=$(mktemp) && wget "https://hndl.urbackup.org/Client/2.5.25/UrBackup%20Client%20Linux%202.5.25.sh" -O $TF && sudo sh $TF; rm -f $TF
```

Once the installation is done, the client will show up on the web interface.

You can specify the Backup location on the server by going to **Settings -> Server** and under **Backup Storage Path** specify the path of the destination Backup folder.

Under **Server URL for client file/backup access/browsing**, specify the server URL along with the port: http://server-ip:55414

***
