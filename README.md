<div align="center">
  <img src="https://raw.githubusercontent.com/dokemon-ng/.github/refs/heads/main/dokemon-logo.png" width="500" alt="Dokémon Logo">
</div>
Dokémon is a friendly GUI for managing Docker Containers. You can manage multiple servers from a single Dokemon instance.

Check https://dokemon.einstein.amsterdam (old: https://dokemon.dev) for more details.

## Quickstart

You can run the below commands to quickly try out Dokémon.

**Note:** Whenever possible, it is recommended that you run Dokémon in a private network and do not expose it to the Internet. In cases where this is not possible, for example when running on a VPS to which you only have public access, you should run Dokémon behind an SSL enabled reverse proxy and use a strong password for maximum security. Refer the next section for sample configuration using Traefik.

    # Create directory to store Dokemon data
    sudo mkdir /dokemondata

    # Run Dokemon
    sudo docker run -p 9090:9090 \
      -v /dokemondata:/data \
      -v /var/run/docker.sock:/var/run/docker.sock \
      --restart unless-stopped \
      --name dokemon-server -d javastraat/dokemon-server:latest

## Using Traefik with LetsEncrypt SSL certificate

This is an example configuration for running Dokémon behind Traefik with LetsEncrypt SSL certificate.

**Note:** This is a sample configuration. Please modify it as per your requirements.

    services:
      traefik:
        image: "traefik:v2.10"
        container_name: "traefik"
        command:
          - "--log.level=DEBUG"
          - "--accesslog=true"
          - "--api.insecure=true"
          - "--providers.docker=true"
          - "--providers.docker.exposedbydefault=false"
          - "--entrypoints.websecure.address=:443"
          - "--certificatesresolvers.dokemon.acme.tlschallenge=true"
          - "--certificatesresolvers.dokemon.acme.email=your.email@example.com"
          - "--certificatesresolvers.dokemon.acme.storage=/letsencrypt/dokemon.json"
        ports:
          - "443:443"
          - "8080:8080"
        volumes:
          - "./letsencrypt:/letsencrypt"
          - "/var/run/docker.sock:/var/run/docker.sock:ro"

      dokemon:
        image: javastraat/dokemon-server:latest
        container_name: dokemon-server
        restart: unless-stopped
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.dokemon.rule=Host(`dokemon.example.com`)"
          - "traefik.http.routers.dokemon.entrypoints=websecure"
          - "traefik.http.routers.dokemon.tls.certresolver=dokemon"
        ports:
          - 9090:9090
        volumes:
          - /dokemondata:/data
          - /var/run/docker.sock:/var/run/docker.sock

In the DNS settings for your domain, add an A record for the _Host_ which you have mentioned in the above config. The A record should point to the public IP address of your virtual machine.

1. Create a file named `compose.yaml` on your server. Copy and paste the above YAML definition into the file. Modify the email and host. Make any other changes as per your requirements.
2. Create .env file in root directory, take .env-example has example and define your own value
3. Run `mkdir ./letsencrypt && mkdir /dokemondata`
4. Run `docker compose up -d`

Open https://dokemon.example.com (substitute your URL here which you entered as Host in the compose.yaml file) in the browser. It can take a few seconds for the SSL certificate to be provisioned. If you get an error related to SSL, please wait for a few moments and then refresh your browser.

## Screenshots

### Manage Multiple Servers
![Alt text](https://github.com/dokemon-ng/dokemon/raw/main/screenshots/screenshot-dokemon-nodes.jpg?raw=true)

### Manage Variables for Different Environments

![Alt text](https://github.com/dokemon-ng/dokemon/raw/main/screenshots/screenshot-dokemon-variables.jpg?raw=true)

### Deploy Compose Projects

![Alt text](https://github.com/dokemon-ng/dokemon/raw/main/screenshots/screenshot-dokemon-compose-up.jpg?raw=true)

### Manage Containers, Images, Volumes, Networks

![Alt text](https://github.com/dokemon-ng/dokemon/raw/main/screenshots/screenshot-dokemon-containers.jpg?raw=true)

## License

This project is [MIT Licensed](LICENSE).

