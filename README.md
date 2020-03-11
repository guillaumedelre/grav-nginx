# grav-nginx
Debian [Buster](https://hub.docker.com/_/debian/) Docker based [Grav](https://getgrav.org) installation using Nginx

**WORK IN PROGRESS**:

- [x] Basic implementation in Docker
- [x] Extend volume mount to local `html` directory
- [x] Allow UID/GID setting within container for host user permissions on mounted volumes
- [x] SSL integration - self generated certs
- [ ] LetsEncrypt documentation

### Build image

```bash
docker-compose build
```

### Usage:

```bash
docker-compose up -d
```

The running site can be found at [http://localhost](http://localhost).

- On first use the user will be redirected to [http://localhost/admin](http://localhost/admin) to register an Admin User for the installation.
![Grav Register Admin User](https://user-images.githubusercontent.com/5332509/27988518-752da2e0-63f1-11e7-8731-9e6e185536c8.png)

**Example 2.** Using `docker-compose.yml` to deploy a locally built container with **(1.)** Local volume mount, **(2.)** SSL using self generated certificates

An example `docker-compose.yml` file is included in the repository to demonstrate how to use the self generated SSL certificates. This is useful for development, but should not be used in prodcution as the generated certificates are not signed by any authority.

```yaml
version: '3.3'                         # Varies based on the installed docker version
services:
  grav-nginx:
    build:                             # Local build definition
      context: ./docker            # Optionally use image: call
      dockerfile: Dockerfile
    container_name: grav
    environment:
      - USE_SSL=true                   # Set to use SSL
      - USE_SELF_GEN_CERT=true         # Set to generate self signed certs
    volumes:
      - "./html:/home/grav/www/html"   # Set to share local volume named html
    ports:
      - "80:80"                        # Serve http on port 80
      - "443:443"                      # Serve https on port 443
```

The user can build and deploy this container as follows:

```bash
$ cd /PATH/TO/grav-nginx/
$ docker-compose build
$ docker-compose up
```
- **NOTE**: Generally the `-d` flag would be used in the above call to daemonize the process

After a few moments the running site can be found at [https://localhost](https://localhost).

- On first use the user will be asked if they wish to proceed to an "unsafe" https site, and when given the OK, redirected to [https://localhost/admin](https://localhost/admin) to register an Admin User for the installation.

![https-ssl-cert-warning](https://user-images.githubusercontent.com/5332509/27988520-7de8ff60-63f1-11e7-9642-0a5a45d9a90c.png)

![https-grav-admin](https://user-images.githubusercontent.com/5332509/27988522-83b1402e-63f1-11e7-9919-5c0d7c7de4f3.png)

All of the grav site related files will now be stored in the local `html` directory of the checked out repository.

```bash
$ ls ./html
CHANGELOG.md       README.md          cache              index.php          robots.txt         vendor
CODE_OF_CONDUCT.md assets             composer.json      local.dev.crt      system             webserver-configs
CONTRIBUTING.md    backup             composer.lock      local.dev.key      tmp
LICENSE.txt        bin                images             logs               user
```

The two files named `local.dev.crt` and `local.dev.key` are the self generated SSL certificate and key pair. These are moved to locations `/etc/ssl/certs/server.crt` and `/etc/ssl/private/server.key` for use by Nginx in the **grav** container by the `docker-entrypoint.sh` script at runtime.
