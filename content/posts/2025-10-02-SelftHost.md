## Mini Self Hosting project
![rp4](/images/rasberrypiMain.jpg)

When starting the internship, one of the things I learned was Self hosting.
The process reffers to Deploying an service locally on your Network and taking full responsibility for your application
I got started with a Rasberry pi 4, an mini Computer with an Debian installed onto it.
On the project i reied on Documentation, Other peoples blogs and very on AI Models like Claude 4.5 Sonnet.
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

You can start the docker container by running: docker compose -f ./compose.yml up 
This will pull the image from docker hub, also will print the logs and give us information on the app.
When started, you will see something simmilar to this on your web browser:
![rp4](/images/rasberrypiGiteap1.png)
This is the configuration, enter the following:
DatabaseSettings: 
    - Database Type: PostgreSQL
    - Host: leave deafault
    - Username: gitea
    - Password: DB_PASSWORD (you entered/or enter default)
    - Database Name: gitea
    - SSL: disable (for now will configure later)
GeneralSettings: Leave everything deafult.

Once done, you will be presented with the main dasboard
![rp4](/images/rasberrypiGiteap2.png)

####  Gitea repository migration from Github