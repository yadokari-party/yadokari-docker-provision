# Provisioning services with Docker
This guide will walk you through the process of setting up Misskey and Backup-Tool using the provided Docker.  

  
## Steps
### Install docker
```bash
curl https://get.docker.com/ | sudo sh
```
   
### Clone this repository
Clone the yadokari-docker-provision repository from GitHub to server:  
```bash
git clone --recursive https://github.com/yadokari-party/yadokari-docker-provision.git
```
   
### Edit configs (Misskey)
Enter any value for `POSTGRES_PASSWORD` and `POSTGRES_USER` (must be the same value as in `default.yml` described below).  
Also, if you are using Cloudflare Tunnel, for `TUNNEL_TOKEN` enter [instructions here](https://qiita.com/mai_llj/items/5485d2a90fe28b76b5aa#cloudflare-argo-tunnel%E3%81%AE%E8%A8%AD%E5%AE%9A) Enter the Cloudflare Tunnel token.  
```
cp ./yadokari-docker-provision/misskey/config/.env.sample ./yadokari-docker-provision/misskey/config/.env
vim ./yadokari-docker-provision/misskey/config/.env
```
  
The same values entered in the `.env` above should be entered in the corresponding sections of the `default.yml`.  
```
vim ./yadokari-docker-provision/misskey/config/default.yml
```
  
### Create docker network
Create a new network to establish communication between Docker containers to PostgreSQL containers.  
This is not required (if you do not do this step, you will need to edit `compose.yaml` separately), but must be done if you use the backup tool described below.  
```bash
sudo docker network create -d bridge --gateway=10.0.0.1 --subnet=10.0.0.0/27 misskey-postgres
```
  
### Launch the service (Misskey)
```bash
cd ./yadokari-docker-provision/misskey/
sudo docker compose --profile proxy --profile tunnel up -d --build
```
  
> [!NOTE]
> Although the `--profile` option is included only for the first boot, Caddy Proxy does not need to be recreated for updates or other tasks, so it can be run without the `--profile` option with `sudo docker compose up -d --build` without the `--profile` option.

With these steps, the construction of the Misskey server is complete.  
By default, inbound traffic is handled through the Caddy Proxy, so it is necessary to establish communication to the appropriate ports.  
This is not described here, as it depends on the environment and cloud you are using.  
  
### Edit configs (Backup-Tool)
This step will be prepared later.
  
  
## Optional Steps
This step does not necessarily need to be done in order to publish the service.  
### Install Tailscale
By deploying Tailscale, remote access VPNs can be easily established. Separate ACLs can be written, allowing zero-trust-based management.  
Follow the steps below to install.  
```bash
curl -fsSL https://tailscale.com/install.sh | sudo sh
sudo tailscale up --ssh --auth-key={{ YOUR_AUTH_KEY }}
```
*You will need to obtain an Auth key from Tailscale's Admin Console. (You can also use `tailscale login`)  
  
  
> [!NOTE]
> If multiple people are required to administer the service, or if another service also belongs to the same tailnet, it is preferable to configure the appropriate ACL settings, since all devices will be able to communicate with each other.  
  
### Access Restriction Settings
Since ssh connections are made via tailscale and web access is made via Cloudflare Tunnel, all inbound communication should be discarded and only specific communication should be allowed.  
Here is an example of a setup using ufw, but it is not limited to this.  
```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 100.64.0.0/10 to any port ssh
sudo ufw reload
sudo service ssh restart
```
  
  
## How to update
Please do this if you need to update your containers due to Misskey updates, infrastructure configuration changes, etc.  
```bash
git pull && sudo docker compose --profile tunnel up -d
```

  
If you need to update Caddy Proxy, please add the `--profiles` option as needed.  
For updates that do not involve restarting Caddy Proxy, the maintenance page will be automatically displayed while the Misskey application container is down.  