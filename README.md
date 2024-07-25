# pypiserver-nginx-cloudflare
Private PyPi - Nginx - Cloudflare SSL

This repository is supposed to create a private PyPI server hosted in Google Compute Engine e2-micro instance (free Tier) with pypi.mydomain.com as hostname and SSL from Cloudflare. You must change hostname to your hostname.

## Installation

## 1. Server setup

- Create Ubuntu e2-micro instance in [Compute Engine](https://console.cloud.google.com/compute), in the Advanced options > Networking, fill in your hostname (ex. pypi.mydomain.com).

- Go to Cloudflare > Website > DNS > Records, then Add A Record to point your hostname to server IP.

## 2. Dependency setup
- Install Docker using this [guide](https://docs.docker.com/engine/install/ubuntu/)  

- Install apache2-utils 

    ```bash
    sudo apt install apache2-utils
    ```


## 3. PyPi server setup
- Clone this repository, and cd into it.
- Create folder to store packages, and create pypiserver user and group and give them access

    ```bash
    sudo addgroup --system --gid 9898  pypiserver
    sudo adduser --uid 9898 --ingroup pypiserver --system --no-create-home pypiserver
    sudo mkdir packages
    sudo chown -R pypiserver:pypiserver packages
    sudo chmod g+s packages
    ```

## 4. Auth setup
- Create .htpasswd file

  ```bash
  httpasswd -sc auth/.httpasswd <username>
  ```

- To add more user,
  ```bash
  httpasswd -sc auth/.httpasswd <second_username>
  ```

## 5. SSL setup
- Go to Cloudflare > Website > SSL/TLS > Origin Server, then create RSA certificate, Fill hostnames with your hostname. Choose certificate validity, then click Create.
- Save Origin Certificate and Private Key as pypi.mydomain.com.pem and pypi.mydomain.com.key, then place it in certs folder.
- Do not forget to adjust hostname in Nginx configuration inside nginx/conf.d/local.conf file.

## 6. Start server
- Start docker 
    
  ```bash
  sudo docker compose up -d
  ```

## Troubleshooting
Site visitors may see untrusted certificate errors if you pause or disable Cloudflare on subdomains that use Origin CA certificates. These certificates only encrypt traffic between Cloudflare and your origin server, not traffic from client browsers to your origin.

Solution: Enable Proxied status in A Records configuration.


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
***It will ask you for username and password.***

## Credits
- https://github.com/node-energy/pypiserver-nginx
