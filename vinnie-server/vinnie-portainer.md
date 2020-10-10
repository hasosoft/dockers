

# Docker
<!-- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04 -->

## Install Docker pre-reqs

`sudo apt install apt-transport-https ca-certificates curl software-properties-common`

## Add docker repo

`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"`

## Install Docker

`sudo apt install docker-ce`

## Check if Docker is running correctly

`sudo systemctl status docker`

## Add user to docker group

`sudo usermod -aG docker ${USER}`

<!-- https://github.com/MatchbookLab/local-persist -->
## Install local-persistant storage driver 

`curl -fsSL https://raw.githubusercontent.com/MatchbookLab/local-persist/master/scripts/install.sh | sudo bash`

## Create storage volumes for Docker
`docker volume create -d local-persist -o mountpoint=/mnt/media/nextclouddata --name=nextclouddata`

`docker volume create -d local-persist -o mountpoint=/mnt/media/pictures --name=pictures`

`docker volume create -d local-persist -o mountpoint=/mnt/media/movies --name=movies`

`docker volume create -d local-persist -o mountpoint=/mnt/media/music --name=music`

Make sure file permissions are set up correctly (nextcloud uses the www-data user)

`chown -R www-data:www-data /mnt/media/nextclouddata`

`chown -R www-data:www-data /mnt/media/pictures`

`chown -R www-data:www-data /mnt/media/movies`

`chown -R www-data:www-data /mnt/media/music`

# Docker Compose
<!-- https://github.com/docker/compose -->

```sudo curl -L https://github.com/docker/compose/releases/download/1.26.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose```

`sudo chmod +x /usr/local/bin/docker-compose`

## download compose files from repository
Navigate to home directory `cd ~`

`git clone -b vinnie-server https://github.com/hasosoft/dockers.git`

Navigate to `dockers/vinnie-server`

# Portainer
<!-- https://clouding.io/kb/en-us/articles/360010398219-Install-Portainer-on-Ubuntu-18-04 -->

## Add storage for portainer config

`docker volume create portainer_data`

## Add network for portainer to connect externally

`docker network create web`

## Start portainer, make sure it works

Navigate to portainer compose folder `cd ~/dockers/vinnie-server/portainer/`

Use compose to run portainer `docker-compose up -d`

Navigate to x.x.x.x:9000 in a browser, set up password, connect portainer to 'Local'

# Traefik
<!-- https://techrevelations.de/2019/11/10/nextcloud-and-traefik-v2/ -->
<!-- https://www.smarthomebeginner.com/traefik-reverse-proxy-tutorial-for-docker/ -->

## Add network for traefik

`docker network create traefik_proxy`

Or go to 'Networks' in portainer and add a network called 'traefik_proxy' while leaving the rest on default

## Port forwarding and Domain

**Make sure** port 80 and/or 443 is forwarded to the IP-address of the *machine running the docker containers*.

**Make sure** the domain is pointing to the *external IP of the router*.

## Start Traefik, make sure it works (portainer)

### Via github

Go to 'Stacks' in portainer, click add stack, select 'git repository', 

fill in **name** `traefikstack`

**repository url** in this case is `https://github.com/hasosoft/dockers/`

**reference** is `refs/heads/vinnie-server` since it's on that 'branch'

**compose path** is `vinnie-server/traefik/docker-compose.yml` since it is in that folder.

**authentication** shouldn't be necessary unless it is in a private repository

### Via compose file

Navigate to `cd ~/dockers/vinnie-server/traefik/`

Copy the contest of the `docker-compose.yml`  file (eg. `cat docker-compose.yml`)

Go to 'Stacks' in portainer, click add stack, and paste the contents of the traefik/docker-compose.yml file into the web editor.

<!-- ## Start Traefik, make sure it works (docker-compose fallback)

**don't use this if you set it up with portainer**

Navigate to `cd ~/dockers/vinnie-server/traefik/`

Use compose to run portainer `docker-compose up -d` -->

## Check if traefik works

Navigate to x.x.x.x:8080 in a browser and see if it works, it should show you a dashboard and should display some charts.

If you go to HTTP -> Routers, it should display `whoami` as the service running on the `websecure` entrypoint. You can verify it's running correctly by navigating to the https domain eg. https://traefik.harro.dev and it should show a text based webpage showing information about the webrequest. SSL certificate should be automatically pulled and applied by Traefik.

# Nextcloud
<!-- https://techrevelations.de/2019/11/10/nextcloud-and-traefik-v2/ -->
<!-- https://github.com/cbirkenbeul/docker-homelab/blob/master/nextcloud/docker-compose.yaml -->

## Add network for nextcloud
`docker network create nextcloud_backend`

## Deploy Nextcloud stack in portainer

### Via github

Go to 'Stacks' in portainer, click add stack, select 'git repository', 

fill in **name** `nextcloudstack`

**repository url** in this case is `https://github.com/hasosoft/dockers/`

**reference** is `refs/heads/vinnie-server` since it's on that 'branch'

**compose path** is `vinnie-server/nextcloud/docker-compose.yml` since it is in that folder.

**authentication** shouldn't be necessary unless it is in a private repository

### Via compose file

Navigate to `cd ~/dockers/vinnie-server/nextcloud/`

Copy the contest of the `docker-compose.yml`  file (eg. `cat docker-compose.yml`)

Go to 'Stacks' in portainer, click add stack, and paste the contents of the nextcloud/docker-compose.yml file into the web editor.

<!-- ## Start Nextcloud (docker-compose fallback)

**don't use this if you set it up with portainer**

Navigate to `cd ~/dockers/vinnie-server/nextcloud/`

Use compose to run portainer `docker-compose up -d` -->

## Setting up nextcloud

If you set `NEXTCLOUD_UPDATE=` to `0` instead of 1 you get the option to set up nextcloud by using the web interface. 

Navigate to either the hostname set in the nextcloud/docker-compose.yml or the local IP address.

Fill in username/password as you like, then expand 'Storage & database' select MySQL/MariaDB and make sure you fill in the following:

- Database user: nextcloud
- Database password: the MYSQL_ROOT_PASSWORD from the mariadb env
- Database name: nextcloud
- localhost: replace with 'nextcloud-mariadb' or whatever your nextcloud database container is called

It should look like this: 

![nextcloud setup](https://github.com/hasosoft/dockers/raw/vinnie-server/vinnie-server/nextcloud/nextcloud-setup.png)

Next if you did the setup from your hostname, you will notice that you get a connection refused error, to fix this the domain has to be added to the ['trusted-domains'](https://docs.nextcloud.com/server/19/admin_manual/installation/installation_wizard.html#trusted-domains) list.

The `docker-compose.yml` file contains an `NEXTCLOUD_TRUSTED_DOMAINS=` entry but that seems to be ignored when running through the nextcloud installer.

To fix this you need to edit `/var/docker/nextcloud/app/config/config.php` and modify 
```  
'trusted_domains' =>
  array (
    0 => 'x.x.x.x',
  ),
```
to contain your hostname and or local ip like

```
  'trusted_domains' =>
  array (
    0 => '10.0.0.201 local ip address',
    1 => 'nextcloud.harro.dev',
  ),

```

This step is also useful when getting general untrusted domain and connection refused errors.

## External Storage in nextcloud

https://docs.nextcloud.com/server/20/admin_manual/configuration_files/external_storage/local.html

Go to nextcloud menu -> apps, find 'External storage support' if it's not in 'Active apps' look in 'Disabled apps' and enable it.

To set it up go to nextcloud menu -> settings -> Administration -> External storages

Name a folder to add, select 'Local' Authentication 'None',
Configuration is where the LOCAL path goes, so the path inside the docker container, in this case they are all mounted to /mnt/media/xxx using the same internal and external path. Click the checkbox to confirm and add more folders as needed.

![external storage options](https://github.com/hasosoft/dockers/raw/vinnie-server/vinnie-server/nextcloud/nextcloud-external-storage.jpg)