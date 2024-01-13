# NeonWhale - DockerLab

This Repository is a collection of ![Docker](https://img.shields.io/badge/Docker%20Compose-0e0b33?style=for-the-badge&logo=docker&logoColor=00ccff) stacks, that I've made for various reasons.

A guide for secure local development (LEC), using either a free or paid domain, can be found [here](#introduction-to-local-development-lec).

Information about how to work with secure local development (SSC) on windows can be found [here](#introduction-to-local-development-ssc).

## Stacks

### [PHP Dev Stack (Local - SSC)](/PHP%20Dev%20Stack%20(Local%20-%20SSC))

#### Services

![NGINX](https://img.shields.io/badge/NGINX-061703?style=for-the-badge&logo=nginx&logoColor=009639)
![PHP](https://img.shields.io/badge/PHP%208.3.1-0a0317?style=for-the-badge&logo=php&logoColor=777BB4)
![MariaDB](https://img.shields.io/badge/MariaDB-011A21?style=for-the-badge&logo=mariadb&logoColor=009BCA)
![PHPMyAdmin](https://img.shields.io/badge/PHPMyAdmin-191824?style=for-the-badge&logo=phpmyadmin&logoColor=6C78AF)

##### Purpose

The purpose of this stack, was to learn more about the nginx server, the content of the configuration files, as well as how a full development configuration for php could look like.
While I have [generated the config files](https://www.digitalocean.com/community/tools/nginx), I've also learned **more** about what they do.

##### Features and Config

-   Nginx Server
    -   Configured for `HTTPS` with `HTTP/2`.
    -   Redirects all `HTTP` requests to `HTTPS`.
    -   Listening for the custom local `$domain => neonspace.com`.
    -   The `Container` path for `NGINX` and `PHP FPM` is `/var/www/$domain`.
    -   The `NGINX` root directory for the files that should **only** be served is pointing to the `Public` directory (Case Sensitive).
    -   Every request is being pushed to / through the `index.php` within the `Public` directory.

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

## Introduction to local development (SSC)

Generating and using a SSC (Self Signed Certificate) can be a bit much, especially if the topic is one you don't know much about. There are plenty of resources out there, in the world wild web, but they don't necessarily have the full information needed to pull it of.

### How To - StepByStep (Windows)

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

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr
```

-   -req => Initiates the creation of a CSR (Certificate Signing Request).
-   -new => Indicates that a new CSR is being created.
-   -newkey rsa:2048 => Creates a new RSA private key of 2048 bits.
-   -nodes => Prevents the encryption of the output key.
-   -keyout domain.key => Specifies the filename to write the newly created private key to.
-   -out domain.csr => Specifies the output filename to write the CSR to.

```bash
openssl x509 -req -days 365 -in domain.csr -signkey domain.key -out domain.crt
```

-   -x509 => Is the OpenSSL utility for displaying and manipulating X.509 certificates.
-   -req => Indicates that a certificate signing request (CSR) is being used.
-   -days 365 => Specifies that the certificate will be valid for 365 days.
-   -in domain.csr => Specifies the input file, which is the CSR.
-   -signkey domain.key => Specifies the file with the private key to use for signing.
-   -out domain.crt => Specifies the output file, which is the certificate.

#### 4. Install the certificate

In order to trust the certificate, as well as remove the warnings and insecurity flags, you have to install it.

1. Open the control panel.
2. Search of `Manage User Certificates`.
3. Navigate to `Trusted Root Certification Authorities`.
4. RightClick `Certificates`.
5. Go to `All Tasks > Import` and follow the Instructions.

#### 5. How to proceed

Now, that you have your files, you can use them `.crt` and `.key` to enable https for your web based projects.
