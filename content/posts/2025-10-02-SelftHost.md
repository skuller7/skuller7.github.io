## Mini Self Hosting project
![rp4](/images/rasberrypiMain.jpg)

When starting the internship, one of the things I learned was Self hosting.
The process reffers to Deploying an service locally on your Network and taking full responsibility for your application
I got started with a Rasberry pi 4, an mini Computer with an Debian installed onto it.
On the project i relied on Documentation, Other peoples blogs and very on AI Models like chatgpt, Gemini & Claude 4.5 Sonnet.
Finnaly not everything will be documented here, cos there is a lot of detail.

## Lab Overview 

In this project,  I wanted to incorporate Self hosting an App like gitea with Cloud services- Azure
The idea was to host an service on Pi4 and communicate directly using an VPN to an Azure VM
Creating an FS backup of PI4 and sending directly to Azure Storage Account Container.
Configuring an Reverse Proxy Server, that will handle all of communication 
Setting up SSL/TLS cerificates and using custom sub Domain names.

## Architecture
Internet → Cloudflare DNS → Azure VM (Traefik) → WireGuard VPN → Raspberry Pi → Gitea

## Project Stages
### Stage 1 - Setting up Gitea 
This was the foundational step in the project, setting up an repository platfrom where i store my repositories.
Ultimately this was also an service which i self hosted.
My personal choise was to host the App on Docker, here is how i configured Gitea service
Link to install Docker+Compose on Debian 12: https://docs.docker.com/engine/install/debian
Link to install Gitea: https://docs.gitea.com/installation/install-with-docker


####  Getting Acces to Raspberry pi 4

We can get access to an pi4, by pluging the device and using ping tool, to check the private ip address of the device
![rp4](/images/rasberrypiSSHp1.png)

To deploy gitea, i created an directory on pi4 where the docker compose file will be located.
Gitea app consist of an: gitea server image + postgresql database
Here is the compose file, to use default credentials create an .env file where everything will be stored or copy the default from gitea link.
We are also using an specific version of gitea version 1.24.5, this will pull the image for both gita and postgresql

```yaml
version: "3.9"
services:
  server:
    image: gitea/gitea:1.24.5
    container_name: gitea-server
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=db:5432
      - GITEA__database__NAME=${POSTGRES_DB}
      - GITEA__database__USER=${POSTGRES_USER}
      - GITEA__database__PASSWD=${POSTGRES_PASSWORD}
      - GITEA__server__SSH_PORT=2221
      - GITEA__server__ROOT_URL=http://${PI_IP:-localhost}:3000
    volumes:
      - gitea-data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "2221:22"
    depends_on:
      - db
    restart: unless-stopped
  db:
    image: postgres:17.5
    container_name: gitea-db
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - gitea-db:/var/lib/postgresql/data
    restart: unless-stopped
volumes:
  gitea-data:
  gitea-db:
```

You can start the docker container by running: ```sh docker compose -f ./compose.yml up ``` 
This will pull the image from docker hub, also will print the logs and give us information on the app.
When started, you will see something simmilar to this on your web browser:
![rp4](/images/rasberrypiGiteap1.png)
This is the configuration, enter the following:
>**DatabaseSettings**: 
    - Database Type: PostgreSQL
    - Host: leave deafault
    - Username: gitea
    - Password: DB_PASSWORD (you entered/or enter default)
    - Database Name: gitea
    - SSL: disable (for now will configure later)
>**GeneralSettings: Leave everything deafult**.

Once done, you will be presented with the main dasboard
![rp4](/images/rasberrypiGiteap2.png)

####  Gitea repository migration from Github

You can migrate your apps repository from github, to gitlab
Not just github there a veriety of platforms you can move from: Codebase, OneDev, AWSCodecommit
To migrate the repository we need some sort of authentication on the github side, using the PAM Tokens
To create an Fine-grained token following this link: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

1. Click on the New migration button and select Github
2. Migrate/Clone From Url
Set the url link of repository to clone to gitea
3. Access Token - PAM
Paste the token you created 
4. Everything else leave as default like on the picture below
![rp4](/images/rasberrypiGiteap3.png)

Your repsitory should be on gitea after the process finishes.
Now lets try to clone the repository to our machine

1. Clone the repository on the Machine
```sh
git clone http://192.168.1.56:3000/admin/subnetCalculator.git
```
2. Create some files using the touch command, stage them and commit using git
```sh
touch file01 file02
git add . 
git commit -m "Added new files"
git push origin main
```

3. Authenticate over the web, and see changes on the Platform.
![rp4](/images/rasberrypiGiteap4.png)

### Stage 2 - Configuring Reverse Proxy on Azure

The plan for this project was to have an proxy server acting as a middleman, for all of our requests
This was possible thanks to Traefik, and it was not very simple setting it up, 
The app is deployed on Azure VM, & you need to make sure to open the right ports using NSG.

```yaml
Here is how the compose file looks like
services:
  traefik:
    image: traefik:v3.5
    command:
      - "--api.dashboard=true"
      - "--api.insecure=false"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--log.level=INFO"
      - "--accesslog=true"

    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    labels:
      # Traefik Dashboard with Basic Auth
      - "traefik.http.routers.api.rule=Host(`traefik.lilixkube.net`)"
      - "traefik.http.routers.api.entrypoints=web"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.middlewares=auth"
      # Basic Auth: admin/password (change this)
      - "traefik.http.middlewares.auth.basicauth.users=<CREDENTIALS>"

  # Test service
  whoami:
    image: "traefik/whoami"
    labels:
      - "traefik.http.routers.whoami.rule=Host(`whoami.lilixkube.net`)"
      - "traefik.http.routers.whoami.entrypoints=web"

  # Gitea proxy service
  gitea-proxy:
    image: "traefik/whoami"  # Just for testing, can remove later
    labels:
      - "traefik.http.routers.gitea.rule=Host(`gitea.lilixkube.net`)"
      - "traefik.http.routers.gitea.entrypoints=web"
      - "traefik.http.routers.gitea.service=gitea-backend"
      - "traefik.http.services.gitea-backend.loadbalancer.server.url=http://172.16.25.2:3000"
      # Optional: Add auth to Gitea too
      # - "traefik.http.routers.gitea.middlewares=auth"
```
One of the most important security mechanisms here is using authentication for traefik.
Run this command on your server, that will generate the password where we will access the traefik
```sh
zUros@GitLabServer:~$ echo $(htpasswd -nb uros sysadmin) | sed -e s/\\$/\\$\\$/g
uros:$$apr1$$a5hbpjaA$$ShMqSFK9cdBdc0oX5vR0u0
zUros@GitLabServer:~$
```
Paste the credentials in traefik labels, every time you need to access the service you will need to authenticate.\

Here is how the NSG supposose to look like, you need to open the ports: 3000 (gitea), 80/443 (HTTP/S), 51820 (Traefik)
![rp4](/images/rasberrypiGiteap5.png)

To only way to access traefik is via domain name or via public ip addresses
There will be documentation later on how to set up Domain names and DNS in general.

