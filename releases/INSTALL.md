# S-UI v1.4.1 Installation Guide

This guide explains how to install this archived/custom build of **S-UI v1.4.1**.

The build contains:

```text
sui
libcronet.so
s-ui-custom-1.4.1-docker-image.tar
BUILD_INFO.txt
```

Recommended installation method: **Docker image**.

## 1. Requirements

Recommended OS:

```text
Ubuntu 22.04 / 24.04
Debian 12
```

Required packages:

```bash
sudo apt update
```

```bash
sudo apt install -y docker.io curl
```

Start and enable Docker:

```bash
sudo systemctl enable --now docker
```

Check Docker:

```bash
sudo docker --version
```

## 2. Install using the Docker image

Go to the directory where the release files are stored:

```bash
cd /path/to/releases
```

Load the Docker image:

```bash
sudo docker load -i s-ui-custom-1.4.1-docker-image.tar
```

Check that the image was loaded:

```bash
sudo docker images | grep s-ui-custom
```

Run S-UI:

```bash
sudo docker run -d --name s-ui --restart unless-stopped --network host -v s-ui-data:/app/db s-ui-custom:1.4.1
```

Check container status:

```bash
sudo docker ps
```

Check logs:

```bash
sudo docker logs -f s-ui
```

## 3. Open the panel

Default web panel:

```text
http://SERVER_IP:2095/app/
```

Default subscription endpoint:

```text
http://SERVER_IP:2096/sub/
```

If you are testing inside WSL, get the WSL IP:

```bash
hostname -I
```

Then open:

```text
http://WSL_IP:2095/app/
```

## 4. Firewall

Allow the panel port:

```bash
sudo ufw allow 2095/tcp
```

Allow the subscription port:

```bash
sudo ufw allow 2096/tcp
```

Reload UFW:

```bash
sudo ufw reload
```

Check status:

```bash
sudo ufw status
```

For a public server, it is safer to restrict access to the admin panel by IP, VPN, or reverse proxy.

## 5. Stop, start, restart

Stop S-UI:

```bash
sudo docker stop s-ui
```

Start S-UI:

```bash
sudo docker start s-ui
```

Restart S-UI:

```bash
sudo docker restart s-ui
```

View logs:

```bash
sudo docker logs -f s-ui
```

## 6. Backup data

The container stores data in the Docker volume:

```text
s-ui-data
```

Create a backup:

```bash
sudo docker run --rm -v s-ui-data:/data -v "$PWD":/backup alpine tar czf /backup/s-ui-data-backup.tar.gz -C /data .
```

Restore a backup:

```bash
sudo docker stop s-ui
```

```bash
sudo docker run --rm -v s-ui-data:/data -v "$PWD":/backup alpine sh -c "rm -rf /data/* && tar xzf /backup/s-ui-data-backup.tar.gz -C /data"
```

```bash
sudo docker start s-ui
```

## 7. Update the container from a new image

Stop and remove the old container:

```bash
sudo docker stop s-ui
```

```bash
sudo docker rm s-ui
```

Load the new image:

```bash
sudo docker load -i s-ui-custom-1.4.1-docker-image.tar
```

Run it again using the same data volume:

```bash
sudo docker run -d --name s-ui --restart unless-stopped --network host -v s-ui-data:/app/db s-ui-custom:1.4.1
```

## 8. Optional: run the standalone binary

Docker is recommended. Use this only if you know what you are doing.

Create installation directory:

```bash
sudo mkdir -p /opt/s-ui
```

Copy files:

```bash
sudo cp sui /opt/s-ui/
```

```bash
sudo cp libcronet.so /opt/s-ui/
```

Make the binary executable:

```bash
sudo chmod +x /opt/s-ui/sui
```

Run manually:

```bash
cd /opt/s-ui
```

```bash
sudo LD_LIBRARY_PATH=/opt/s-ui ./sui
```

## 9. Troubleshooting

Check whether the panel is listening:

```bash
sudo ss -lntp | grep -E '2095|2096'
```

Check container logs:

```bash
sudo docker logs --tail=100 s-ui
```

Check if the panel responds locally:

```bash
curl -I http://127.0.0.1:2095/app/
```

A `307 Temporary Redirect` to `/app/login` means the panel is responding correctly.

## 10. Notes

This is an archived/custom build of S-UI.

Frontend source was restored from:

```text
ClashConnectRules/s-ui-frontend
commit f65f58e
```

Use at your own risk.
