# pypiserver-nginx-cloudflare

Private PyPi - Nginx - Cloudflare SSL

This repository is supposed to create a private PyPI server hosted in Google Compute Engine e2-micro instance (free Tier) with pypi.mydomain.com as hostname and SSL from Cloudflare. You must change hostname to your hostname.

## Installation

### 1. Server setup

- Create Ubuntu e2-micro instance in [Compute Engine](https://console.cloud.google.com/compute), in the Advanced options > Networking, fill in your hostname (ex. pypi.mydomain.com).

- Go to Cloudflare > Website > DNS > Records, then Add A Record to point your hostname to server IP.

### 2. Dependency setup

- Install Docker using this [guide](https://docs.docker.com/engine/install/ubuntu/)

- Install apache2-utils

  ```bash
  sudo apt install apache2-utils
  ```

### 3. PyPi server setup

- Clone this repository, and cd into it.
- Create folder to store packages, and create pypiserver user and group and give them access

  ```bash
  sudo addgroup --system --gid 9898  pypiserver
  sudo adduser --uid 9898 --ingroup pypiserver --system --no-create-home pypiserver
  sudo mkdir packages
  sudo chown -R pypiserver:pypiserver packages
  sudo chmod g+s packages
  ```

### 4. Auth setup

- Create .htpasswd file

```bash
# Ensure file exists without wiping
touch nginx/auth/.htpasswd

# Add/Update user using strong Bcrypt encryption
htpasswd -B nginx/auth/.htpasswd <username>
```

- To add more user,

```bash
htpasswd -B nginx/auth/.htpasswd <second_username>
```

### 5. SSL setup

- Go to Cloudflare > Website > SSL/TLS > Origin Server, then create RSA certificate, Fill hostnames with your hostname. Choose certificate validity, then click Create.
- Save Origin Certificate and Private Key as pypi.mydomain.com.pem and pypi.mydomain.com.key, then place it in certs folder.
- Do not forget to adjust hostname in Nginx configuration inside nginx/conf.d/local.conf file.

### 7. Start PyPi server

- Start docker
  ```bash
  sudo docker compose up -d
  ```
  Docker will pull the latest image, build, and then start pypiserver and nginx containers.

### 8. Stop PyPi server

- Stop pypiserver and nginx container

  ```bash
  sudo docker stop pypiserver nginx
  ```

## Update

For updating the image, use the following steps:

### 1. Stop and remove the containers

```bash
sudo docker compose down
```

### 2. Pulling latest image, build, and start the containers.

```bash
sudo docker compose up --pull always --build -d
```

### 3. Remove unused old image

```bash
sudo docker image prune -f
```

## Troubleshooting

- Certificate error on client side

Site visitors may see untrusted certificate errors if you pause or disable Cloudflare on subdomains that use Origin CA certificates. These certificates only encrypt traffic between Cloudflare and your origin server, not traffic from client browsers to your origin.

Solution: Enable Proxied status in A Records configuration.

- PyPi server not running after VM reboot

Sometimes, VM automatically restarted if they are terminated for non-user-initiated reasons (maintenance event, hardware failure, software failure and so on). In Google Compute Engine, we can add startup script that will run when your instance boots up or restarts.

```bash
#! /bin/bash

cd /home/<your_username>/pypiserver-nginx-cloudflare

sudo docker compose up -d
```

## Usage

### Poetry

#### Build

Update version number in pyproject.toml, then

```bash
poetry build -f wheel
```

#### Publish package

Make sure poetry is configured to have access to that PyPI.

```bash
poetry config repositories.myrepo https://pypi.mydomain.com/
poetry config http-basic.myrepo <username> <password>
```

Publish the package

```bash
poetry publish -r myrepo
```

#### Install package

##### Using Poetry

- Add your repository

  ```bash
  poetry source add --priority=supplemental myrepo https://pypi.mydomain.com/simple/
  ```

- Configure your credentials

  ```bash
  poetry config http-basic.myrepo <username> <password>
  ```

- Install package
  ```bash
  poetry add --source myrepo <package_name>
  ```

### Pip

- Install package

```bash
pip install -f https://pypi.mydomain.com/packages <package-name>
```

**_It will ask you for username and password._**

## Credits

- https://github.com/node-energy/pypiserver-nginx
