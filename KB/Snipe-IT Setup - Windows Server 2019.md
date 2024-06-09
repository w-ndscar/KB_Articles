
*This article follows this amazing video: [# SNIPE-IT Installation On Windows Server 2016](https://www.youtube.com/watch?v=kxGGQutvktM&list=PLuB-nSPBO_bM_dly7SMWP7l08iqL3nrml)*

#### Prerequisites:

- IIS Server + CGI
- [PHP](https://windows.php.net/download/)
- [C++ Visual Studio Redistributable (2012)](https://www.microsoft.com/en-au/download/details.aspx?id=30679) 
- [C++ Visual Studio Redistributable (2015)](https://www.microsoft.com/en-au/download/details.aspx?id=48145)
- [PHP Manager for IIS](https://www.iis.net/downloads/community/2018/05/php-manager-150-for-iis-10)
- [MySQL/MariaDB](https://downloads.mariadb.org/)
- [Snipe-IT](https://snipeitapp.com/download) (Ofcourse)

#### Let's Begin

Download and setup the above mentioned prerequisites by click on them.


### Fixing the folder permissions

C:/inetpub/wwwroot/snipe-it/public
Right click on Uploads -> Properties -> Security -> Edit -> Add
IUSR with Full Control

C:/inetpub/wwwroot/snipe-it/bootstrap
Right click on cache -> Properties -> Security -> Edit -> Add
IUSR with Full Control

C:/inetpub/wwwroot/snipe-it
Right click on storage -> Properties -> Security -> Edit -> Add
IUSR with Modify



