References: [Zabbix - Open Source, Self Hosted Server, Network, and Device monitoring system with power!](https://www.youtube.com/watch?v=ec2G1PeLS5k&list=PLAldzxXBXnFAjtPWmFyZwZP05lXjfJTkA&index=10&t=1330s)

### Objective:
***
To install Zabbix monitoring server as a Docker Container and monitor the hosts using the Zabbix agent
***

### Prerequisites
***
Docker and Docker compose are needed
***

### Cloning the Zabbix repo
***
Let's start by cloning the [Zabbix repository](https://github.com/zabbix/zabbix-docker) on Github using the command:
```shell
git clone https://github.com/zabbix/zabbix-docker
```

Then get into the zabbix-docker directory and copy the desired docker-compose file into docker-compose.yaml

For an instance we'll use the docker-compose_v3_ubuntu_mysql_latest.yaml file:
```shell
cp docker-compose_v3_ubuntu_mysql_latest.yaml docker-compose.yaml
```
***
### Setting up the environment variables
***
Change to the directory: env_vars inside the zabbix-docker directory
*Note: The files inside the env_vars directory are hidden.*
Change the MYSQL_PASSWORD and  MYSQL_ROOT_PASSWORD to something strong
Edit the file: .env_web. For ZBX_SERVER_NAME, specify a name for the Zabbix Server
Edit the file: .env_web_service. For ZBX_TIMEOUT, set the value to 15 (15 minutes)

### Deploying the Zabbix Server
***
Navigate to the zabbix-docker directory and use the following command to deploy the Zabbix Server:
```shell
sudo docker-compose up -d
```
***
