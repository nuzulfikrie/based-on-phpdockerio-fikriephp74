# This is based on PHPDocker.io generated environment

# Add to your project

Simply, unzip the file into your project, this will create `docker-compose.yml` on the root of your project and a folder
named `phpdocker` containing nginx and php-fpm config for it.

Ensure the webserver config on `phpdocker/nginx/nginx.conf` is correct for your project. PHPDocker.io will have
customised this file according to the front controller location relative to the docker-compose file you chose on the
generator (by default `public/index.php`).But in my case, it will run from a folder called `webroot` as I were developing for Cakephp 3.

Note: you may place the files elsewhere in your project. Make sure you modify the locations for the php-fpm dockerfile,
the php.ini overrides and nginx config on `docker-compose.yml` if you do so.

# How to run

Dependencies:

- docker. See [https://docs.docker.com/engine/installation](https://docs.docker.com/engine/installation)
- docker-compose. See [docs.docker.com/compose/install](https://docs.docker.com/compose/install/)

Once you're done, simply `cd` to your project and run `docker-compose up -d`. This will initialise and start all the
containers, then leave them running in the background.

## Services exposed outside your environment

You can access your application via **`localhost`**. Mailhog and nginx both respond to any hostname, in case you want to
add your own hostname on your `/etc/hosts`

| Service               | Address outside containers                |
| --------------------- | ----------------------------------------- |
| Webserver             | [localhost:16999](http://localhost:16999) |
| Mailhog web interface | [localhost:17000](http://localhost:17000) |
| MariaDB               | **host:** `localhost`; **port:** `17002`  |

## additional notes for cakephp and laravel

### cakephp, connecting to database that is hosted on the host machine

- change the host to `host.docker.internal` in your `config/app.php` file for the database connection.
- change the host to `host.docker.internal` in your `config/app_local.php` file for the database connection.

## Note To connect to a MariaDB instance running on your host machine from a Docker container, you have a few options depending on your OS:

### Linux

1. **Use host network mode**: In your `docker-compose.yml`, you can set the network mode for the service that needs to connect to the host's MariaDB.

   ```yaml
   services:
     your-service:
       network_mode: "host"
   ```

   Note: This only works on Linux.

2. **Use a special DNS name**: Docker for Linux provides a special DNS name `host.docker.internal` that resolves to the internal IP address used by the host.

   So, in your application configuration, you would set the MariaDB hostname to `host.docker.internal` and the port to whatever port MariaDB is running on (usually `3306`).

### macOS and Windows

1. **Use special DNS name**: Docker for Windows and for Mac provides a special DNS name `host.docker.internal` that resolves to the internal IP address used by the host. This can be used for connecting to services running on the host machine.

### Example Configuration

Here is how you might configure a PHP application to connect to a MariaDB running on the host, using environment variables:

```php
$dbhost = getenv('DB_HOST') ?: 'host.docker.internal';
$dbport = getenv('DB_PORT') ?: 3306;
$dbname = getenv('DB_NAME') ?: 'mydatabase';
$dbuser = getenv('DB_USER') ?: 'root';
$dbpass = getenv('DB_PASS') ?: 'rootpassword';

$conn = new mysqli($dbhost, $dbuser, $dbpass, $dbname, $dbport);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
```

In this example, if environment variables are not set, it defaults to `host.docker.internal` for the host and `3306` for the port.

Remember to modify your `docker-compose.yml` to match your preferred approach and don't forget to rebuild your containers:

```bash
docker-compose down
docker-compose up --build
```

Make sure that your MariaDB on the host machine is set up to allow connections from the IP ranges used by Docker containers, and that you're using a user/password that has the necessary permissions.

## laravel connecting to database that is hosted on the host machine

- change the host to `host.docker.internal` in your `.env` file for the database connection.

## using the webserver cli and ping the database that is hosted on the host machine

- run `docker-compose exec webserver bash` to open a shell on the webserver container.
- run `ping host.docker.internal` to ping the host machine.

## Hosts within your environment

You'll need to configure your application to use any services you enabled:

| Service        | Hostname | Port number    |
| -------------- | -------- | -------------- |
| php-fpm        | php-fpm  | 9000           |
| MariaDB        | mariadb  | 3306 (default) |
| Redis          | redis    | 6379 (default) |
| SMTP (Mailhog) | mailhog  | 1025 (default) |

# Docker compose cheatsheet

**Note:** you need to cd first to where your docker-compose.yml file lives.

- Start containers in the background: `docker-compose up -d`
- Start containers on the foreground: `docker-compose up`. You will see a stream of logs for every container running.
  ctrl+c stops containers.
- Stop containers: `docker-compose stop`
- Kill containers: `docker-compose kill`
- View container logs: `docker-compose logs` for all containers or `docker-compose logs SERVICE_NAME` for the logs of
  all containers in `SERVICE_NAME`.
- Execute command inside of container: `docker-compose exec SERVICE_NAME COMMAND` where `COMMAND` is whatever you want
  to run. Examples:
  - Shell into the PHP container, `docker-compose exec php-fpm bash`
  - Run symfony console, `docker-compose exec php-fpm bin/console`
  - Open a mysql shell, `docker-compose exec mysql mysql -uroot -pCHOSEN_ROOT_PASSWORD`

# Application file permissions

As in all server environments, your application needs the correct file permissions to work properly. You can change the
files throughout the container, so you won't care if the user exists or has the same ID on your host.

`docker-compose exec php-fpm chown -R www-data:www-data /application/public`

# Recommendations

It's hard to avoid file permission issues when fiddling about with containers due to the fact that, from your OS point
of view, any files created within the container are owned by the process that runs the docker engine (this is usually
root). Different OS will also have different problems, for instance you can run stuff in containers
using `docker exec -it -u $(id -u):$(id -g) CONTAINER_NAME COMMAND` to force your current user ID into the process, but
this will only work if your host OS is Linux, not mac. Follow a couple of simple rules and save yourself a world of
hurt.

- Run composer outside of the php container, as doing so would install all your dependencies owned by `root` within your
  vendor folder.
- Run commands (ie Symfony's console, or Laravel's artisan) straight inside of your container. You can easily open a
  shell as described above and do your thing from there.

# Simple basic Xdebug configuration with integration to PHPStorm

## Xdebug 2

To configure **Xdebug 2** you need add these lines in php-fpm/php-ini-overrides.ini:

### For linux:

```
xdebug.remote_enable = 1
xdebug.remote_connect_back = 1
xdebug.remote_autostart = 1
```

### For macOS and Windows:

```
xdebug.remote_enable = 1
xdebug.remote_host = host.docker.internal
xdebug.remote_autostart = 1
```

## Xdebug 3

To configure **Xdebug 3** you need add these lines in php-fpm/php-ini-overrides.ini:

### For linux:

```
xdebug.mode=debug
xdebug.discover_client_host=true
xdebug.start_with_request=yes
xdebug.client_port=9000
```

### For macOS and Windows:

```
xdebug.mode = debug
xdebug.client_host = host.docker.internal
xdebug.start_with_request = yes
```

## Add the section “environment” to the php-fpm service in docker-compose.yml:

```
environment:
  PHP_IDE_CONFIG: "serverName=Docker"
```

### Create a server configuration in PHPStorm:

- In PHPStorm open Preferences | Languages & Frameworks | PHP | Servers
- Add new server
- The “Name” field should be the same as the parameter “serverName” value in “environment” in docker-compose.yml (i.e. _
  Docker_ in the example above)
- A value of the "port" field should be the same as first port(before a colon) in "webserver" service in
  docker-compose.yml
- Select "Use path mappings" and set mappings between a path to your project on a host system and the Docker container.
- Finally, add “Xdebug helper” extension in your browser, set breakpoints and start debugging

### Create a launch.json for visual studio code

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Docker",
      "type": "php",
      "request": "launch",
      "port": 9000,
      // Server Remote Path -> Local Project Path
      "pathMappings": {
        "/application/webroot": "${workspaceRoot}/"
      }
    }
  ]
}
```
