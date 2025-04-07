#### Objective: To setup Samba AD-DC Server and File Share

***

## Install Samba and its dependencies

To make samba to server AD services, we have to install some dependencies

Run the following command to install samba and its dependencies
```
apt-get install acl attr samba winbind libpam-winbind libnss-winbind krb5-config krb5-user dnsutils python3-setproctitle
```

There will be three prompts from now on
- For the first prompt of `Default Kerberos version 5 realm`, type in the Domain Name in CAPITAL LETTERS
- For the second prompt of  `Kerberos servers for your realm`, type in the same Domain Name in lowercase
- For the third prompt of `Administrative server for your Kerberos realm`, type in the same Domain Name in lowercase


A samba DC will usually serve as a domain NTP server. So, install NTP using,
```
apt-get install ntp
```

***
## Provisioning Samba AD

Before provisioning the samba AD, rename the smb.conf file as the provisioning script will automatically create one for us.

To remove it, run the command,
```
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

Stop the relevant services by using the following commands
```
sudo systemctl stop samba.service smbd.service nmbd.service winbind.service && sudo systemctl disable samba.service smbd.service nmbd.service winbind.service
```

The Samba AD can be provisioned in both interactive and non-interactive modes.

For interactive mode, run the command,
```
samba-tool domain provision --use-rfc2307 --interactive
```
Then fill out the details as it prompts.

For non-interactive mode, modify the following command according to your needs and run it
```
samba-tool domain provision --server-role=dc --use-rfc2307 --dns-backend=SAMBA_INTERNAL --realm=SAMDOM.EXAMPLE.COM --domain=SAMDOM --adminpass=Passw0rd
```

Modify the `--realm`, `--domain` and `--dns-backend` according to your needs

There are two options for DNS Backend: `SAMBA_INTERNAL` and `BIND9_DLZ`. Refer to [The SAMBA AD DNS Back Ends](https://wiki.samba.org/index.php/The_Samba_AD_DNS_Back_Ends) for more information. 

If you specify `BIND9_DLZ` for DNS Backend, continue with [[#Setting up the BIND DNS Server]]

If you specify the `--dns-backend` to be `SAMBA_INTERNAL`, **SKIP** the "Setting up the BIND DNS Server" Section. The internal DNS server (`SAMBA_INTERNAL`) is only able to resolve the Active Directory (AD) DNS zones. To enable recursive queries of other zones, set the DNS forwarder parameter to something like 1.1.1.1 or 8.8.8.8
```
sudo nano /etc/named.conf
```

```named.conf.options
options{
...
...
forwarders { 8.8.8.8; 1.1.1.1; };

};
```


***

## Setting up the BIND DNS Server

Most of it is referred from this [samba-bind-config](https://wiki.samba.org/index.php/Setting_up_a_BIND_DNS_Server) documentation

Install BIND packages by running
```
sudo apt-get install -y bind9 bind9utils
```

We are going to setup the BIND DNS Server. We have 4 Bind9 Files to work with.
```
/etc/bind/named.conf
/etc/bind/named.conf.options
/etc/bind/named.conf.local
/etc/bind/named.conf.default-zones
```

Of these, only two need to be configured.

The first file `/etc/bind/named.conf`, Shouldn't need modification, as it just contains links to the other named.conf files: 
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

The second file `/etc/bind/named.conf.options`, is the one you need to configure for your Active Directory and to setup default ACL's for Bind9. 

Rename the `named.conf.options` file
```
sudo mv /etc/bind/named.conf.options /etc/bind/named.conf.options.bak
```

and create a new one
```
sudo nano /etc/bind/named.conf.options
```

and copy the following contents
```
// Managing acls
acl internals { 127.0.0.0/8; 192.168.0.0/24; };

options {
      directory "/var/cache/bind";
      version "Go Away 0.0.7";
      notify no;
      empty-zones-enable no;
      auth-nxdomain yes;
      forwarders { 8.8.8.8; 1.1.1.1; };
      allow-transfer { none; };

      dnssec-validation no;
      // Removed with BIND 9.18:
      // dnssec-enable no;
      
      // If you only use IPv4. 
      listen-on-v6 { none; };
      // listen on these ipnumbers. 
      listen-on port 53 { 192.168.0.6; 127.0.0.1; ::1; };

      // Added Per Debian buster Bind9. 
      // Due to : resolver: info: resolver priming query complete messages in the logs. 
      // See: https://gitlab.isc.org/isc-projects/bind9/commit/4a827494618e776a78b413d863bc23badd14ea42
      minimal-responses yes;

      //  Add any subnets or hosts you want to allow to use this DNS server
      allow-query { "internals";  };
      allow-query-cache { "internals"; };

      //  Add any subnets or hosts you want to allow to use recursive queries
      recursion yes;
      allow-recursion {  "internals"; };

      // https://wiki.samba.org/index.php/Dns-backend_bind
      // DNS dynamic updates via Kerberos (optional, but recommended)
      // ONE of the following lines should be enabled AFTER you provision or join a DC with bind9_dlz 
      // or AFTER upgrading your dns from internal to bind9_dlz 
      // Before Samba 4.9.0
      // tkey-gssapi-keytab "/var/lib/samba/private/dns.keytab";
      // From Samba 4.9.0 ( You will need to run samba_upgradedns if upgrading your Samba version. ) 
      tkey-gssapi-keytab "/var/lib/samba/bind-dns/dns.keytab";

  };
```

You need to specify your subnet here by replacing **192.168.0.0/24** -> `acl internals { 127.0.0.0/8; 192.168.0.0/24; };` 

Then specify the Samba domain server IP here by replacing **192.168.0.6** -> `listen-on port 53 { 192.168.0.6; 127.0.0.1; ::1; };`

The third file `/etc/bind/named.conf.local`, just needs the addition of one line, to link in another file provided by Samba: 
```
include "/var/lib/samba/bind-dns/named.conf";
```

Then start the BIND Server using,
```
sudo systemctl start named
```

If there is an error while starting the named service, try running the following upgrade command and start the named service

```
samba_upgradedns --dns-backend=BIND9_DLZ
```

#### Configure the DNS Resolver

Edit the /etc/resolv.conf and add the lines
```
search samdom.example.com
nameserver 10.99.0.1
```
Modify the **search** with FQDN (DC name and domain name) 
Modify the nameserver with the Samba DC's IP

#### Copy the Kerberos config

Copy the auto-created Kerberos config file to the `/etc/` location with the following command
```
sudo mv /etc/krb5.conf /etc/krb5.conf.initial
sudo cp /var/lib/samba/private/krb5.conf /etc/
```


***

## Create samba-ad-dc service (Only if samba.service does not exist)

Create a new file named samba-ad-dc.service at `/etc/systemd/system/`

```
sudo nano /etc/systemd/system/samba-ad-dc.service
```

and paste the following contents

```
[Unit]
Description=Samba Active Directory Domain Controller
After=network.target remove-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/samba/sbin/samba -D
PIDFile=/usr/local/samba/var/run/samba.pid
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

Then run,
```
sudo systemctl daemon-reload && sudo systemctl start named
```

***

## Reload and Start

Reload the daemon by
```
sudo systemctl daemon-reload
```

Then enable the samba services by running
```
sudo systemctl enable samba.service smbd.service nmbd.service 
```

Then start Samba with
```
sudo systemctl start samba.service
```

***

## Final Steps - Part 1 (Optional)

Let's test the Samba AD DC

#### Verify the File Server
```
smbclient -L localhost -N
```

With authentication
```
smbclient //localhost/netlogon -UAdministrator -c 'ls'
```

#### Verify DNS
```
host -t SRV _ldap._tcp.samdom.example.com.
host -t SRV _kerberos._udp.samdom.example.com.
host -t A dc1.samdom.example.com.
host -t PTR 10.99.0.1
```

#### Verify Kerberos
```
kinit administrator
```

***

## Final Steps - Part 2 (Optional)

Raise the domain function level from 2008_R2 to 2012_R2. First, modify the smb.conf with the following,

```
sudo nano /etc/samba/smb.conf
```

```smb.conf
[global]
...
...
ad dc functional level = 2012_R2
```

Then restart the samba and smb
```
sudo systemctl restart samba smb
```

Raise the Domain Level with
```
sudo samba-tool domain level raise --domain-level=2012_R2
```

Raise the Forest Level with
```
sudo samba-tool domain level raise --forest-level=2012_R2
```

Then restart the samba and smb again
```
sudo systemctl restart samba smb
```

Take a look at the following table for Samba version and Functional levels

| Functional Level | Included in Samba Version |
| :--------------- | :------------------------ |
| 2016             | 4.20 and later            |
| 2012_R2          | 4.19 and later            |
| 2012             | 4.19 and later            |
| 2008_R2          | 4.0 and later             |
| 2008             | 4.0 and later             |
| 2003             | 4.0 and later             |
***

## Reference Links

 - [Setting up Samba as an AD DC](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)
 - [Setting up a BIND DNS Server](https://wiki.samba.org/index.php/Setting_up_a_BIND_DNS_Server)
 -  [The SAMBA AD DNS Back Ends](https://wiki.samba.org/index.php/The_Samba_AD_DNS_Back_Ends)
 - [Raising the Domain Functional Levels](https://wiki.samba.org/index.php/Raising_the_Functional_Levels)

***
