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
    available via localhost. We can access it through a temporary forward SSH
    tunnel.

The containers share the same [network](docker-compose.yml#L63-L64) to be able
to communicate with each other.

The full setup further requires a reverse-proxy configuration for
[Apache2](extras/apache2-site.conf) or Nginx, and a [systemd system
service](extras/invoiceninja.service) to automatically start and restart the
container union.
