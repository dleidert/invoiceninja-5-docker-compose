# invoiceninja-docker-compose - Sample docker-compose Config for Invoice Ninja

This is an example setup to run a self-hosted Invoice Ninja instance via Docker
container. The idea is to run everything inside a container and to not expose
any involved services to the public, except for the web interface via a reverse
proxy supporting HTTPS.

As a bonus, the SQL database can be accessed directly via phpmyadmin. This
shouldn't be necessary, but I have experienced situations where this became
necessary. This interface is also not for public use.

** Caution: Services not intended for public use should NEVER be made available
publicly. **

## Design

This [multi-container setup](docker-compose.yml) requires a running server with
a recent operating system supporting Docker. The containers involved are:

  * [`invoiceninja_web`](docker-compose.yml#L7-L22): web-server to provide HTTP
    access to the Invoice Ninja application; it's interface will be made
    available on `localhost` only and can be accessed through a reverse proxy
    providing HTTPS. It can be tweaked via its [own config](vhost.conf) file.

  * [`invoiceninja_app`](docker-compose.yml#L24-L38): The application's backend
    container, provided by the Invoice Ninja application. It will store its
    files in a persistent storage and share the public files with the
    web-server container. No ports will be published.

  * [`invoiceninja_db`](docker-compose.yml#L40-L48): The official MySQL 8
    Docker container running the database for the instance. It contains the
    data entered through the app's web interface. The data is stored in a
    persistent mount point.

  * [`invoiceninja_dbadmin`](docker-compose.yml#L50-L61): The official
    phpMyAdmin container running a web interface to the database. This
    container is completely **optional**. Its interface is not made public, but
    available via `localhost.` We can access it through a temporary forward SSH
    tunnel.

The containers share the same [network](docker-compose.yml#L63-L64) to be able
to communicate with each other.

The full setup further requires a reverse-proxy configuration for
[Apache2](extras/apache2-site.conf) or Nginx, and a [systemd system
service](extras/invoiceninja.service) to automatically start and restart the
container union.

## Required Software

This software is required:

  * `docker.io` | `docker-ce` | `docker`
  * optional: `docker-compose` (one can use `docker compose` command as well)
  * `apache2` | `nginx-light` | `nginx`

The docker version shipped in most distributions might already be fine. It
definitely is for Debian Bullseye, for example. There is not really a reason to
get the docker CE packages.

If SysVinit is used instead of systemd, then the service file contains every
necessary to write an init script based on it.

## Configuration

Although one can change the [`docker-compose.yml`](docker-compose.yml) file,
most configuration takes place in the [`.env`](.env) file. There is an example
configuration in the [`.env.example`](.env.example) file.

Let's go though each item:

  * `INVOICE_NINJA_HOME`: This is used in the [`docker-compose.yml`](docker-compose.yml) file to define the local path where the database and the application files should be stored outside the container in a persistent location. The variable in the systemd service file overwrites the variable defined here.

## Accessing phpMyAdmin

It is intended to not expose the port to access the phpMyAdmin service to the
public. To use it, we can create an SSH forward tunnel from our host to the
server running the container. A forward tunnel is basically a tunnel, that
starts on a local IP and port and forwards all packetsi from there to a remote
IP and port. So, if phpMyAdmin is available on the remote server on `localhost`
port `8000`, we can do:

```
ssh -N -L localhost:8000:localhost:8000 <remote server>
```

then we can access phpMyAdmin locally via <http://localhost:8000/> as long as
the tunnel exists, and we can view, change, and backup the remote database.

## Issues and FAQS

** I: The interface shows only a few languages and currencies and misses most
after the first login. **

Open <http://your.domain.tld/update?secret=secret> in the browser. Of course
replace the placeholder here by the real domain.

Thanks to
<https://forum.invoiceninja.com/t/missing-languages-currencies/12026/3>.

** I: There is an update available **

If we run the systemd service, we just restart the server:

```
sudo systemctl restart invoiceninja.service
```

If we run the containers manually, then we basically have to shut the
containers down, pull the updates, and start them again:

```
docker-compose down
docker-compose pull --include-deps
docker-compose up (-d)
```


