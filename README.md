## SuiteCRM Docker Development Environment

##Pre requisites

#### Installation
**Check if you have docker installed**

```
docker -v
```

**Install docker if you get command not found**

[For Ubuntu 14:04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started#how-to-install-docker)

[For Ubuntu 16:04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04#step-1-—-installing-docker)

[Remove Sudo Requirement](https://docs.docker.com/engine/installation/linux/linux-postinstall/)


**Check if you have docker-compose installed**

```
docker-compose -v
```

**Install docker-compose if you get command not found**

```
$ curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

``` 
chmod +x /usr/local/bin/docker-compose

```

### How it works

**Working Locally**

Your local machine no longer works as a web server that responsibility is now dockers
you work as you normally do in the var/www folder but when changes are made they are
synced onto the docker server.

**GIT**

You use git as you normally would on your local machine.

### Using the SuiteCRM Environment

Setup Environment Variables
```
cp .env-dist .env

```

```
nano .env
```

Set the Mysql Root Password Details
* MYSQL_ROOT_PASSWORD
*
And change any environment settings like php version or mysql version by uncommenting/commenting out the versions.

Removing the environment
```
docker-compose rm -v
```

Build The Environment
```
docker-compose build --no-cache 
```
Running The Environment
```
docker-compose up
```
Running The Environment in the background
```
docker-compose up -d
```
View list of running containers
```
docker ps
```
Enter a container
```
docker exec -it <container name> bash
```
To switch between PHP/MySql versions edit .env file and run 
(if it is the first time you switch version it may take a while to install)
```
docker-compose up --build
```
Remove all existing containers
```
sudo docker rm -f $(sudo docker ps)
```
Remove all existing images
```
sudo docker rmi -f $(sudo docker images)
```

### How to access applications

These are the default ports if you have changed any in the .env file some may not work.

[PHP 5.3](http://localhost:9053) - http://localhost:9053
[PHP 5.4](http://localhost:9054) - http://localhost:9054
[PHP 5.5](http://localhost:9055) - http://localhost:9055
[PHP 5.6](http://localhost:9056) - http://localhost:9056
[PHP 7.0](http://localhost:9070) - http://localhost:9070
[PHP 7.1](http://localhost:9071) - http://localhost:9071

[PhpMyAdmin](http://localhost:9002) - http://localhost:9002

[MailCatcher](http://localhost:1080) - http://localhost:1080

### How to access selenium
Install a vnc viewer
```
apt-get install remmina
```

Use vnc viewer to connect to selenium node.

protocol: VNC
server: localhost:5900
quality: best
color: 24bit
options:
* show remove cursor
* disable clipboard sync
* disable server input
* view only
password: secret

### Xdebug

To setup xdebug we just need to modify some settings in PHP Storm.

* Go to settings -> Php -> servers 
* Enter the host as 0.0.0.0 and the port as 9001 if you are using the default.
* Save settings.

### Debuging Problems using Logs and running commands outside container. 

There is no need to gaining access to the container to get access to logs, most are piped to the docker logs. 

```
 sudo docker logs --tail 5 --follow <ContainerName>
```

To see only the php errors
```
mkdir /var/www/html/log/
tail -f $(docker logs -f <ContainerName> | grep "PHP" > /var/www/html/log/error_<ContainerName>.log)
```
Example:
```
tail -f $(docker logs -f suitedocker_php_5_5_1 | grep "PHP" > /var/www/html/log/error_suitedocker_php_5_5_1.log)
```

Highlight With colors:
Add the following to the end of your command
```
| awk '/Notice/ {print "\n\033[34m"} '/Warning/' {print "\n\033[33m"} '/Fatal/' {print "\n\033[31m"} {print $0} '
```

Example:
Run in a termimal:
tail -f $(docker logs -f suitedocker_php_5_5_1 | grep "PHP" > /var/www/html/log/error_suitedocker_php_5_5_1.log)

Run in an other terminal:
tail -f /var/www/html/log/error_suitedocker_php_5_5_1.log | awk '/Notice/ {print "\n\033[34m"} '/Warning/' {print "\n\033[33m"} '/Fatal/' {print "\n\033[31m"} {print $0} '

If for any reason you need to run a command in the docker container you can use the following.
 
```
sudo docker exec <ContainerName> sh -c ' service apache2 status'
```

Example of getting all databases exported from the mysql container. 

```
docker exec <ContainerName> sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql
```

Example of how to run the microsoft sql commands
```
docker exec -it <container_id|container_name> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P <your_password>
```

## Installing SuiteCRM (MySQL)
database: suitecrm
hostname: mysql
username: root
password: (see your .env file)

## Installing SuiteCRM (Microsoft SQL)

You need to create a database first. A good tool for this is dbeaver http://dbeaver.jkiss.org/ or the intergrated
database tools in your IDE.

database: suitecrm
hostname: mssql instance: <leave blank>
username: sa
password: (see your .env file)

### Troubleshooting

ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?

Run the command with sudo or [Remove Sudo Requirement](https://docs.docker.com/engine/installation/linux/linux-postinstall/)

### Working with Database

docker exec suitedocker_mysql_1 sh -c 'exec mysqldump <db-name> -uroot -proot' > /var/www/<path-to-dump>.sql
