# NeonWhale - DockerLab

This Repository is a collection of ![Docker](https://img.shields.io/badge/Docker%20Compose-0e0b33?style=for-the-badge&logo=docker&logoColor=00ccff) stacks, that I've made for various reasons.

Methods for obtaining a Certificate, in thise case for local development, can be found [here](#methods-for-obtaining-certificates).

## Stacks

-   [PHP Dev Stack (Local - SSC)](#php-dev-stack-local---ssc)
-   [PHP Dev Stack (Local - LEC)](#php-dev-stack-local---lec)

### [PHP Dev Stack (Local - SSC)](</PHP%20Dev%20Stack%20(Local%20-%20SSC)>)

#### Services

![NGINX](https://img.shields.io/badge/NGINX-061703?style=for-the-badge&logo=nginx&logoColor=009639)
![PHP](https://img.shields.io/badge/PHP%208.3.1-0a0317?style=for-the-badge&logo=php&logoColor=777BB4)
![MariaDB](https://img.shields.io/badge/MariaDB-011A21?style=for-the-badge&logo=mariadb&logoColor=009BCA)
![PHPMyAdmin](https://img.shields.io/badge/PHPMyAdmin-191824?style=for-the-badge&logo=phpmyadmin&logoColor=6C78AF)

##### Purpose

The purpose of this `Stack`, was to learn more about the nginx server, the content of the configuration files, as well as how a full development configuration for php could look like.
While I have [generated the config files](https://www.digitalocean.com/community/tools/nginx), I've also learned **more** about what they do.

##### Features and Config

-   Nginx Server

    -   Configured for `HTTPS` with `HTTP/2`.
    -   Redirects all `HTTP` requests to `HTTPS`.
    -   Listening for the custom local `domain => your_domain.com`.
    -   The `Container` path for `NGINX` and `PHP FPM` is `/var/www/domain`.
    -   The `NGINX` root directory for the files that should **only** be served is pointing to the `Public` directory (Case Sensitive).

-   PHP FPM

    -   A `php.ini-development` and `php.ini-production` template.
        -   Both have the same values, as the original templates, except for `[extension_dir, pdo_mysql]`.
        -   `php.ini-development` has XDebug enabled and configured.
    -   A `php.ini` file. (Currently set for development).
    -   PDO_MySQL installed and enabled.
    -   XDebug installed and enabled.

-   PHPMyAdmin

    -   Used for easy database management.
    -   `HTTPS` only.

-   MariaDB
    -   Using the `create_tables.sql` from PHPMyAdmin, the `PHPMyAdmin Database` is being populated.

### [PHP Dev Stack (Local - LEC)](</PHP%20Dev%20Stack%20(Local%20-%20LEC)>)

#### Services

![NGINXProxyManager](https://img.shields.io/badge/NGINX%20Proxy%20Manager-29100A?style=for-the-badge&logo=nginxproxymanager&logoColor=F15833)
![NGINX](https://img.shields.io/badge/NGINX-061703?style=for-the-badge&logo=nginx&logoColor=009639)
![PHP](https://img.shields.io/badge/PHP%208.3.1-0a0317?style=for-the-badge&logo=php&logoColor=777BB4)
![MariaDB](https://img.shields.io/badge/MariaDB-011A21?style=for-the-badge&logo=mariadb&logoColor=009BCA)
![PHPMyAdmin](https://img.shields.io/badge/PHPMyAdmin-191824?style=for-the-badge&logo=phpmyadmin&logoColor=6C78AF)

##### Purpose

This `Stack` is a demo for how to use the NGINXProxyManager, in order to obtain a wildcard Certificate, allowing for a trusted local environment. (Though this can also be used for external access).

##### Features and Config

Mainly using the NGINXProxyManager.

The other services can be found [here](#features-and-config).

Difference? The SSL directories are gone, which also means that they are not being copied or added.
On top of that the Dockerfile and NGINX Server config no longer have SSL directives.

## Methods for obtaining Certificates

-   [Self Signed Certificates (SSC)](#introduction-to-ssc--how-to---stepbystep-windows)
-   [Signed Wildcard Certificates using LetsEnCrypt, with a free option! (LEC)](#introduction-to-lec--how-to---stepbystep-docker)

### Introduction to (SSC) | How To - StepByStep (Windows)

Generating and using a SSC (Self Signed Certificate) can be a bit much, especially if the topic is one you don't know much about. There are plenty of resources out there, in the world wild web, but they don't necessarily have the full information needed to pull it of.

In order to be able to locally map an IP-Address to a "Domain", you need to edit the hosts file with elevated privileges.

#### 1. Navigate to `c:\Windows\System32\Drivers\etc\` and open the `hosts` file.

```ps
cd c:\Windows\System32\Drivers\etc\

notepad hosts
```

This is how the `hosts` file could look like, if you have docker installed:

```txt
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
# Added by Docker Desktop
192.168.178.157 host.docker.internal
192.168.178.157 gateway.docker.internal
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal
# End of section
```

#### 2. Create your custom mapping

You can just go to the end of the `hosts` file and for example write the following:

```txt
# Custom Defined Mappings

127.0.0.1	your_domain.com
```

```diff
! Save and close the hosts file and powershell window, otherwise the changes might not take effect!
```

#### 3. Get your certificates

For this step, it is time to get a temporary `container` that allows us to generate what we need.

There are many ways to do this and probably even better ones, but I'm sticking to something, where I can safely say that it works.

We will create an ubuntu container, update the package list, install openssl and generate our files.

```ps
cd LOCATION_WHERE_YOU_WANT_TO_STORE_YOUR_CERTS
```

```ps
docker run -it --rm -v ${pwd}:/home/certs -w /home/certs ubuntu
```

-   docker run => Creates a `Container` from an image.
-   -it => Launches that `Container` in interactive mode. (Especially **necessary if** the **container has no job** to do).
-   --rm => Removes the `Container` after exiting or stopping it.
-   -v => Either creates a volume or binds / mounts a host path to a `container` path. (**Bind** in this case).
-   -w => Sets the work directory, which will be the path you start in.
-   image:tag => The image you want to use for the `container`. (**ubuntu:latest** in this case).

If the openssl commands below do not work with this docker run command, try installing it.

```bash
apt update && apt install openssl
```

Due to some issues, that can arise when failing to fill out specific questions upon generating the CSR, I've decided to update this entry with an example config file.

Just replace everything in <> with the actual value and save it as `openssl.cnf`, within the same directory you've launched your `docker run` command in.

```cnf
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = <Country Name (2 letter code)>
ST = <State or Province Name (full name)>
L = <Locality Name (eg, city)>
O = <Organization Name (eg, company)>
OU = <Organizational Unit Name (eg, section)>
CN = <Common Name (e.g., the domain name)>

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = <your-domain-name>
```

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr -config openssl.cnf
```

-   -req => Initiates the creation of a CSR (Certificate Signing Request).
-   -new => Indicates that a new CSR is being created.
-   -newkey rsa:2048 => Creates a new RSA private key of 2048 bits.
-   -nodes => Prevents the encryption of the output key.
-   -keyout domain.key => Specifies the filename to write the newly created private key to.
-   -out domain.csr => Specifies the output filename to write the CSR to.
-   -config => This specifies the configuration file to use, which includes the details for the SAN(s). (Subject Alternative Name(s)).

```bash
openssl x509 -req -days 365 -in domain.csr -signkey domain.key -out domain.crt -extensions v3_req -extfile openssl.cnf
```

-   -x509 => Is the OpenSSL utility for displaying and manipulating X.509 certificates.
-   -req => Indicates that a certificate signing request (CSR) is being used.
-   -days 365 => Specifies that the certificate will be valid for 365 days.
-   -in domain.csr => Specifies the input file, which is the CSR.
-   -signkey domain.key => Specifies the file with the private key to use for signing.
-   -out domain.crt => Specifies the output file, which is the certificate.
-   -extensions v3_req => This specifies the extensions to use for the certificate, which are defined in the configuration file.
-   -extfile => This specifies the configuration file that contains the extension details, including the SANs.

#### 4. Install the certificate

In order to trust the certificate, as well as remove the warnings and insecurity flags, you have to install it.

1. Open the control panel.
2. Search of `Manage User Certificates`.
3. Navigate to `Trusted Root Certification Authorities`.
4. RightClick `Certificates`.
5. Go to `All Tasks > Import` and follow the Instructions.

#### 5. How to proceed

Now, that you have your files, you can use them `.crt` and `.key` to enable https for your web based projects.

### Introduction to (LEC) | How To - StepByStep (Docker)

Ever wanted a dynamic way, to give everything you serve a subdomain without requesting a certificate for every subdomain?
Well, your in luck! The answer is (LEC) LetsEnCrypt.
In this section I'm showing you a way (or two), to generate one certificate and use it for an x amount of services.

Granted, you will need to own a Domain, but don't be discouraged, because while there are `TLD` you can try on sites like [NameCheap](https://www.namecheap.com/), [CloudFlare](https://www.cloudflare.com/) or [DigitalOcean](https://www.digitalocean.com/), you can also use a free service called [DuckDNS](https://www.duckdns.org/), which will provide you a free subdomain you can branch of.

I was wondering if there is a way to get a certificate for an IP address, so that local development can continue in a free manner, but then I finally figured out, how to make a SSC (Self Signed Certificate) work under windows.

But then I learned that (LEC) Let'sEncrypt has more than just the HTTP challenge, which makes this entire thing possible. You can make a challenge, that verifies you as the Domain owner, allowing for a wildcard certificate.

#### 1. Own a Domain

Whether you own a Domain from [DuckDNS](https://www.duckdns.org/), or working with a paid one, the only difference would be, that one a real one you would have to specify a `CNAME` entry where the `alias` would be `*` and point tp `@` or `root`.

The IP of your `A Type` entry points to whatever your use case is. If you only want to access whatever you develop on your local machine, then you can point it to `127.0.0.1`. If you want to be able to access your services on your `LAN`, then you should use your local IP Address.

I went with buying a Domain from [NameCheap](https://www.namecheap.com/) and configuring it with the DNS from [CloudFlare](https://www.cloudflare.com/).
But in the following steps, I'll use [DuckDNS](https://www.duckdns.org/), because I want everyone, to be able to follow along.

#### 2. Create a stack with NginxProxyManager

There are other solutions out there, but this is for now, the simplest one.

```diff
! You don't have to / should bind any ports of a service, that runs or is connected to this Stack.

+ Only the ProxyManager needs to expose ports.
```

Persistent volume variant:

```docker
version: '3.8'
name: "web_stack_with_proxy"

services:
  nginx_proxy_manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx_proxy_manager
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - proxy_manager_data:/data
      - proxy_manager_letsencrypt:/etc/letsencrypt

volumes:
  proxy_manager_data:
  proxy_manager_letsencrypt:
```

Shared directories (Bind / Mount), for when you want to use either the `Certificates` or `Config Data` for example in another `Container` or `Stack`:

```docker
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

#### 3. Configure the NginxProxyManager

This is fairly simple. You'll need to log in, set your first user and then you are ready to do the certificate request.

`DEFAULT_USER: admin@example.com`

`DEFAULT_PASSWORD: changeme`

#### 4. Obtain your Certificate

Navigate to the `SSL Certificates` panel and then click on `Add SSL Certificate`.

![Navigate to SSL Certificates](https://github.com/DrNeonsy/DrNeonsy/assets/118444485/511cc8c6-6460-4f6f-a96c-ab251c05a9bb)

Now, enter your Domains, which could look like `your_domain.tld` and `*.your_domain.tld`. After that you must check the `Use a DNS Challenge`.

![Domains](https://github.com/DrNeonsy/DrNeonsy/assets/118444485/1fc14849-2b91-4d9b-afae-a59b73f6c840)

Choose your DNS provider, fill out whatever is required, change the `Propagation` time to `120s`, because the challenge can fail, if the time is too short, even though you did everything right and `Agree the TOS`.

![DNS Provider](https://github.com/DrNeonsy/DrNeonsy/assets/118444485/43c61c31-9e33-4fba-aebb-a69177d92e60)

You can now download that certificate, if you need to.

#### 5. Proxy your other Stack Services

Let's imagine you have an `NGINX Service` and it only listens to port 80 and does not use SSL. Well seeing that we are using a `Proxy`, we are no longer directly talking to that (other) `NGINX`, so we can just do the following:

![Proxy Host](https://github.com/DrNeonsy/DrNeonsy/assets/118444485/e84d8735-6ce1-440e-acae-d38fced9afeb)

-   Domain Names => Set's the mapping of what you type in the URL.
-   Scheme => Whether your target is HTTP or HTTPS.
-   Forward Hostname / IP => The IP of the HOST Client you want to reach. (Forward to).

You can also define a suitable subdomain, just like this:

![Subdomain](https://github.com/DrNeonsy/DrNeonsy/assets/118444485/88020246-1b94-4c88-b285-c3d93fd954ef)

Now, in order to secure this forwarding, we have to choose the `Certificate` to use for the forwarding, as well as whether we want to force `SSL`, enable `HTTP/2` and more.

![Select Certificate](https://github.com/DrNeonsy/DrNeonsy/assets/118444485/f6c829e8-bb46-402b-b809-12e3e06b3b80)
