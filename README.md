<p align="center">
    <a href="http://github.com/luisaveiro/docker-reverse-proxy">
        <img 
            src="./images/programming.svg" 
            alt="programming" 
            width="50%">
    </a>
</p>

<h4 align="center">
    Local reverse proxy for Docker containers powered by Caddy.
</h4>

<p align="center">
    <a href="#about">About</a> •
    <a href="#disclaimer">Disclaimer</a> •
    <a href="#getting-started">Getting Started</a> •
    <a href="#download">Download</a> •
    <a href="#how-to-use">How To Use</a> •
    <a href="#faq">FAQ</a>
</p>
<p align="center">
    <a href="#useful-tips">Useful Tips</a> •
    <a href="#changelog">Changelog</a> •
    <a href="#contributing">Contributing</a> •
    <a href="#security-vulnerabilities">Security Vulnerabilities</a> •
    <a href="#credits">Credits</a> •
    <a href="#license">License</a>
</p>

## About

This repository offers a simple approach to have [Caddy](https://caddyserver.com/) 
Server has a local reverse proxy for your [Docker](https://www.docker.com/) containers.

**Why would you need a local reverse proxy?**  
With a hosts file, you can only map hostnames to IP addresses. Hosts file does 
not support mapping ports with IP addresses. You are unable to map multiple 
containers to hostnames.

As a developer, you might be working on multiple Docker-based projects. We have 
all experienced the Docker container port binding failure message - 
*Bind for 0.0.0.0:80 failed: port is already allocated.*

Or you are working on a multi-tenant application that requires tenants to log 
in on their subdomain. Developing locally can be challenging with this 
requirement.

Having a reverse proxy you can assign domains or subdomains to your Docker 
containers without the need to worry about port binding failures or develop 
locally with a multi-tenant subdomain application.

## Disclaimer

**Please note:** ***Docker Reverse Proxy*** is not affiliated with Stack Holdings 
GmbH or Docker, Inc and is not an official product from either company. 

"Caddy" is a registered trademark of Stack Holdings GmbH.
"Docker" is a registered trademark of Docker, Inc.

***Docker Reverse Proxy*** has been developed to run Caddy in a local Docker 
environment. If you wish to use Caddy on your system, you can follow the follow 
the [Installation Guide](https://caddyserver.com/docs/install).

## Getting Started

You will need to make sure your system meets the following prerequisites:

- Docker Engine >= 19.03.0

This repository utilizes [Docker](https://www.docker.com/) to run Caddy. So, 
before using ***Docker Reverse Proxy***, make sure you have Docker installed 
on your system.

## Download

You can clone the latest version of ***Docker Reverse Proxy*** repository for 
macOS, Linux and Windows.

```bash
# Clone this repository.
$ git clone git@github.com:luisaveiro/docker-reverse-proxy.git --branch main --single-branch
```

## How To Use

There are a few steps you need to follow before you can have Caddy set up and 
running as a reverse proxy for your Docker containers. I have outline the steps 
you would need to take to get started.

#### 1. <ins>Update your hosts file</ins>

You will need to modify your `/etc/hosts` file to include the hostnames entries.
The hostname is the domain or subdomain you wish to direct traffic for a Docker 
container. 

The structure of the hostname entry must be your loopback IP Address 
(`127.0.0.1`) and domain/subdomain. For example:

```
127.0.0.1 domain.local
```

**Please note:** the hosts file **does not support wildcard** (`*`) for domains 
or subdomains. You will need to have an entry for each domain and subdomain you 
wish to use for the reverse proxy in your hosts file.

#### 2. <ins>Create a Caddyfile</ins>

Once you have added your domain/subdomain entry to your hosts file, you will 
need to create a `Caddyfile`. The Caddyfile is a convenient Caddy configuration 
format for humans. 

***Docker Reverse Proxy*** includes a `Caddyfile.example` file in the config 
directory to help you get started. You can run the following command in the 
terminal to create your Caddyfile file.

```bash
# Create Caddyfile from Caddyfile.example.
$ cp config/Caddyfile.example config/Caddyfile
```

#### 3. <ins>Add your site blocks to your Caddyfile</ins>

Caddy supports various configurations, you can follow the 
[Caddy Documentation](https://caddyserver.com/docs/caddyfile) to learn more 
about the various configurations supported by Caddyfile.

This readme will focus on the Caddyfile reverse proxy configuration 
(*with minimum configuration*). You will need to create a site block in the 
Caddyfile to match the domain entry of your hosts file. 

The `Caddyfile.example` file has an example of the structure of the site block.

```ini
# Site block
<hostname>:80 {
  reverse_proxy <container-name>:<port>
}
```

Let's cover the the Caddyfile's structure:

- **Site block**  
The site block defines the Caddy configuration for a given site address. You 
can identify a site block by the curly braces.

- **Site address**  
The site address is the domain/subdomain entry which matches your hostname in 
your hosts file. Unlike your hosts file, Caddyfile supports port mapping.

- **Directive**  
A directive is the Caddy instruction. Caddy has a variety of 
[directives](https://caddyserver.com/docs/caddyfile/directives). This is readme 
will focus on the [reverse_proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy) 
directive.

- **Argument**  
The reverse_proxy argument is the upstream which Caddy will proxy. In our use 
case the upstream will be the Docker container name and the exposed port. 

Below is an example of a configured site block:

```ini
# Personal website
website.local:80 {
  reverse_proxy website_app:8080
}
```

#### 4. <ins>Start Caddy Docker container</ins>

After you have configured your Caddyfile, you can start Caddy Docker container.
***Docker Reverse Proxy*** includes a `docker-compose.yml` file with Caddy 
pre-configured. You can run the following command:

```bash
# Start Caddy Docker container
$ docker-compose up

# Or start Caddy Docker container detached mode
$ docker-compose up -d
```

Docker will create the Caddy container which is called `docker-reverse-proxy`. 
The container will be attached to a network called `reverse_proxy`. 

If you want to change the container name or network name, you can create a 
DotEnv file and override the Docker Compose variables. Below is an example of 
the DotEnv variables.

```ini
# Docker container
CONTAINER_NAME="docker-reverse-proxy"

# Docker container network
NETWORK_NAME="reverse_proxy"
```

***Docker Reverse Proxy*** includes a `.env.example` file to get you started. 
You can run the following command in the terminal to create your DotEnv file.

```bash
# Create .env from .env.example.
$ cp .env.example .env
```

#### 5. <ins>Attach Docker containers to Caddy network</ins>

Once Caddy container is up and running, you will need to configure your Docker 
containers by attaching the **reverse_proxy** network to your container(s), 
assign a name to the container and expose the port that Caddy will proxy.

I have outlined the necessary configuration below both for Docker Compose and 
Docker CLI approach.

**Docker Compose**

In your Docker Compose file you need to define **reverse_proxy** as an external 
network. For each services you want to access via Caddy reverse proxy, you will 
need to add **reverse_proxy** as an attached network and define your exposed 
port.

Below I have provided an example of a Docker Compose file configured to match 
the example site block in the Caddyfile.

##### **Caddyfile**

```ini
# Personal website
website.local:80 {
  reverse_proxy website_app:8080
}
```

##### **Docker Compose**

```yaml
version: '3.8'

services:
  website:
    # Container name should be the same as in the Caddyfile.
    container_name: website_app
    image: nginx:alpine
    restart: unless-stopped
    # Expose the mapped port that Caddy will proxy.
    expose:
      - 8080
    # Add reverse_proxy as attached network.
    networks:
      - reverse_proxy
    volumes:
      - ./src:/usr/share/nginx/html

networks:
  # Add reverse_proxy as an external network.
  reverse_proxy:
    external: true
```

**Docker CLI**

If you don't use Docker Compose, I have included an example of Docker CLI to 
start a container with the necessary configurations.

```bash
$ docker run --rm --name=website_app --expose 8080 --network=reverse_proxy nginx:alpine
```

## FAQ

**Q:** Are you planning to add local HTTPS support?  
**A:** Yes, this feature is on my roadmap. I'm researching how to implement 
[mkcert](https://github.com/FiloSottile/mkcert) without the need to install 
additional dependencies.

**Q:** If I change the Caddyfile, do I need to restart the Docker container?  
**A:** According to Caddy Docker Hub page, Caddy comes with a `caddy reload` 
command. You can run the following command to reload Caddy configuration:

```
$ docker exec -w /etc/caddy docker-reverse-proxy caddy reload
```

The working directory is set to `/etc/caddy` so Caddy can find your Caddyfile 
without additional arguments.

## Useful Tips

[VSCode Caddyfile Syntax](https://marketplace.visualstudio.com/items?itemName=zamerick.vscode-caddyfile-syntax) 
is an extension for VSCode that adds syntax highlighting for Caddyfiles.

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Contributing

I encourage you to contribute to ***Docker Reverse Proxy***! Contributions are 
what make the open source community such an amazing place to be learn, inspire, 
and create. Any contributions you make are **greatly appreciated**.

Please check out the [contributing to Docker Reverse Proxy guide](.github/CONTRIBUTING.md) 
for guidelines about how to 
proceed.

## Security Vulnerabilities

Trying to report a possible security vulnerability in ***Docker Reverse Proxy***? 
Please check out our [security policy](.github/SECURITY.md) for guidelines 
about how to proceed.

## Credits

The illustration used in the project is from [unDraw (created by Katerina Limpitsouni)](https://undraw.co/). 
All product names, logos, brands, trademarks and registered trademarks are 
property of their respective owners.

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.

---

<p align="center">
  <a href="http://github.com/luisaveiro" target="_blank">GitHub</a> •
  <a href="https://uk.linkedin.com/in/luisaveiro" target="_blank">LinkedIn</a> •
  <a href="https://twitter.com/luisdeaveiro" target="_blank">Twitter</a>
</p>
