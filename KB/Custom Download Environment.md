### Objective: 
***
To create a custom environment where users will open a browser and download a file and access it on the SMB Share (Samba). The browser session will be destroyed after use. 
***

### Setup KASM Workspace
***
[KASM](https://kasmweb.com/) is used for streaming containerized apps/desktops to end-users. KASM uses docker and VNC. 
We are going to deploy Chromium docker container inside KASM.

To install KASM, open a terminal and do the following:

```bash
cd /tmp
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.14.0.3a7abb.tar.gz
tar -xf kasm_release_1.14.0.3a7abb.tar.gz
sudo bash kasm_release/install.sh
```

The installation might take a while as it sets up docker, nginx, VNC and other stuff. Once the installation is done the credentials of the Admin, User and Database will be displayed. Make sure to copy them and then change them accordingly.

Log into the Web Application running on port 443 at **https://<WEBAPP_SERVER>**
The Default usernames are admin@kasm.local and user@kasm.local
***
### Create a Workspace
***
We are going to create a Workspace containing Chromium docker container. To do so, head over to Workspaces at the Left pane and select Workspaces Registry.

In the Workspaces registry search for Chromium and install it. Then select the Installed Workspaces Tab at the top to list the installed workspaces.
***

### Create a persistent data storage
***
We are going to create a persistent data storage as the user downloaded files should be available even after the session is destroyed. 
Select the right side pointing arrow '>' at the Chromium workspace and click on edit

Scroll down to Volume mapping and add the following:

```json
{
  "path/on/the/hostmachine": {
    "bind": "/home/kasm-user/Downloads",
    "mode": "rw",
    "uid": 1000,
    "gid": 1000,
    "required": true
  }
}
```

This script will bind the **Downloads** folder inside the container to the specified path on the host machine.
***

### Share the specified path using Samba

We are going to share the downloaded files, so that the Windows user will be able to access them.
To install samba, first run an update on the repo: `sudo apt update`
Then install samba using the command:
```
sudo apt install samba
```

After the installation, we are going to edit the `smb.conf` file to specify the properties of the file share. Create a copy of the existing smb.conf file by entering the command:
```
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

To edit the file, we are going to use nano. 

```
sudo nano /etc/samba/smb.conf
```

We are going to specify the sharing properties in a separate file: `shares.conf` and include it on `smb.conf`. Enter the following and also modify according to your needs:

```smb.conf
[global]
server string = AppServer
workgroup = WORKGROUP
security = user
map to guest = Bad User
name resolve order = bcast host
include = /etc/samba/shares.conf
```

Now enter
```
sudo nano /etc/samba/shares.conf
```

and enter the following and modify accordingly:

```shares
[External Downloads]
path = /mnt/UUID/foldername
force user = username
force group = smbgroup
create mask = 0664
force create mode = 0664
directory mask = 0775
force directory mode = 0775
browsable = yes
writable = no
read only = yes
```

Notice that the path is a mounted path as its a separate NTFS partition containing the shared folder. The username is the user that will be accessing the share.

Create a group named smbgroup by using the command
```
sudo groupadd smbgroup
```

Create a newuser named smbuser. Make sure the smbuser is a system user with no home folder and with the shell access denied and is a member of the smbgroup

```
sudo useradd --system --no-create-home -s /bin/false --group smbgroup smbuser
```

Then create a smb user set an smb password for the user with the command
```
sudo smbpasswd -a smbuser
```

Now the users would be able to download the files on the Chromium Workspace and access them via SMB Share on Windows.

We can further add an Antivirus and scan the SMB shared folder if a file/folder changes occur. Refer to the [[ClamAV with On-Access Scanning on Linux]] doc for more info